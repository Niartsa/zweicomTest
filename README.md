# Zweicom Test
Este es un pipeline de Jenkins el cual contiene una integración con componentes AWS para la contrucción del proyecto (Un proyecto de .Net en ete caso.

Pasos:

1) Primero se obtiene el número de build de un bucket (S3) para darle un número único al build. De esta manera se puede garantoizar que dentro del proyecto se tendrá un número íunico por build sin importar el branch que se esté construyendo. Luego de obtener el número, se actualiza el archivo sumando +1 al número de build.

2) Se injecta este número en las variables de entorno del proyecto.

3) Se procede a clonar el proyecto de Github.

4) Se crea un archivo de propiedades el cual tendra parámetros que también seran injectados como variables de entorno del proyecto.

5) Se procede a construir el proyecto utilizando MSBuild tool.

Consideraciones:

* Este pipeline esta montado sobre un SIC Jenkins que está montado sobre un servidor Linux, por lo que fue necesario la creación de una instancia EC2 basada en Windows para poder contruirlo utilizando herramientas Microsoft. Es por eso que se utiliza un nodo llamado "ec2node" el cuál es Windows based.

* Iba a colocarle integración con Artifactory, de hecho sería lo más lógico debido a como está pensado la arquitectura.

* Falto la implemntación de envío de correos en caso de falla debido a tiempo.

* Como se puede ver, el archivo JenkinsFile está basado en Groovy.
