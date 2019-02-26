#!/usr/bin/env groovy

node ('ec2node') {
	stage('Get build number') {
		def number = "0"
		int buildNumber = 0
		String fileName = env.JOB_NAME.split("/")[0]
		withAWS(credentials:'CredencialAWS') {
			s3Download(file:'NextBuildNumber/' + fileName + '.txt', bucket: 's3BucketName', path:'NextBuildNumber/' + fileName + '.txt', force:true)
		}
		if (fileExists('NextBuildNumber/' + fileName + '.txt')) {
			number = readFile('NextBuildNumber/' + fileName + '.txt').trim()
		}
		if (number.isInteger()) {
			buildNumber = number as Integer
		}
		writeFile file: 'NextBuildNumber/' + fileName + '.txt', text: "" + (buildNumber + 1)
		withAWS(credentials:'CredencialAWS') {
			s3Upload(file:'NextBuildNumber/' + fileName + '.txt', bucket:'s3BucketName', path:'NextBuildNumber/' + fileName + '.txt')
		}
		currentBuild.displayName = "" + buildNumber
	}
	stage('Inject build number') {
		load "NextBuildNumber/CEDM.txt"
	}
	stage('Clone ' + env.BRANCH_NAME + ' branch to Agent Node.') {
		checkout([
			$class: 'GitSCM',
			branches: [[name: env.BRANCH_NAME]],
			extensions: [[$class: 'WipeWorkspace']],
			userRemoteConfigs: [[credentialsId: 'GitHubCredential', url: 'git@github.com/Niartsa/zweicomTest.git']]
		])
	}
	stage('Create Build.properties file') {
		def DBName = env.JOB_NAME.split("/")[0]
		def BranchName = env.JOB_NAME.split("/")[1]
		def BldNum = currentBuild.displayName
		bat(returnStatus: true, script: "call>build.properties")
		bat(returnStatus: true, script: "echo DBName='${DBName}'>>build.properties")
		bat(returnStatus: true, script: "echo BranchName='${BranchName}'>>build.properties")
		bat(returnStatus: true, script: "echo BldNum='${BldNum}'>>build.properties")
	}
	stage('Inject build.properties file') {
		load "build.properties"
	}
	stage('Build Project') {
		def msbuild =  "C:/\"Program Files (x86)\"/MSBuild/14.0/Bin/amd64/MSBuild.exe"
		def exitStatus = bat(returnStatus: true, script: "${msbuild} CEDM.sln /t:rebuild /p:configuration=Release /p:OctoPackEnforceAddingFiles=True /p:CmdLineInMemoryStorage=True /p:RunOctoPack=true /p:OctoPackPackageVersion=1.0.0.${BldNum} /p:OctoPackAppendToPackageId=${BranchName} /p:BUILD_VERSION=1.0.0.${BldNum}")
		if (exitStatus != 0) {
			currentBuild.result = 'FAILURE'
			error 'build failed'
		}
	}
}
