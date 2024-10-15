# desafio6_devops
Trabajo desafio 6  - Jenkins y Apache

**Instalar Jenkins en VM1** 

#Chequeo si tengo Java instalado

java --version

#El paso siguiente es agregar la clave GPG para garantizar la seguridad del paquete de Jenkins a instalar, para ello ejecutamos los comandos: 
"curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null"

#y añadimos el repositorio

"signed-by=/usr/share/keyrings/jenkins-keyring.asc" 

#esto le indica al sistema que los paquetes de este repositorio deben ser verificados utilizando la clave GPG que se guardó anteriormente. 

#instalamos Jenkins:

apt install jenkins -y

#como el resultado es "El paquete 'jenkins' no tiene un candidato para la instalación" , procedemos a instalarlo manualmente:

"wget https://pkg.jenkins.io/debian-stable/binary/jenkins_2.414.2_all.deb"
"dpkg -i jenkins_2.414.2_all.deb"

#Al tener errores con las dependencias ejecuto "apt-get install -f"

Luego de corregir los errores de dependencias procedo a iniciar jenkins con "systmectl"

"systemctl start jenkins" 
"systemctl enable jenkins"

#Queda corriendo en el puerto 8080, ingresamos en un navegador a localhost:8080/login .

#luego lo que tenemos que hacer es debloquear Jenkins, obteniendo la clave generada automaticamente: “cat /var/lib/jenkins/secrets/initialAdminPassword”

#Ingresamos y verificamos que está correctamente instalado, instalo algunos plugins y “Jenkins is ready!”

** Exponer Jenkins con Ngrok **

#Vamos a descargar Ngrok: “wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-stable-linux-amd64.zip”

#Extraemos el archivo “ngrok” del .tgz y lo movemos al directorio /usr/local/bin/

#Nos registramos en la página de Ngrok y obtenemos un token de autenticación para poder usarlo:

ngrok authtoken xxxxxxxxxxxxxxxxxx (token obtenido en el registro)

#Para iniciar ngrok, tenemos que indicar el puerto que vamos a exponer, en este caso es el 8080:

Digitamos "ngrok http 8080"

#En lo indicado en “Forwarding”, es la URL publica donde queda expuesto Jenkins, lo comprobamos ingresando desde un navegador

**Instalar OpensSSH en VM2:**

Creamos otra VM con UbuntuServer, 

#Ejecuto el comando “dpkg -l | grep openssh-server” para ver si está instalado:

#Y para verificar si está corriendo, ejecuto “sudo systemctl status ssh”:

#Si no estuviese instalado el paquete, debería haber ejecutado el comando: “sudo apt install openssh-server -y” 

**Generar y configurar llaves SSH:**

#Primero debemos generar la llave en lo que será la primera VM (VM1 que tiene Jenkins instalado), ejecuto el comando “ssh-keygen -t rsa -b 4096”, para tipo de clave RSA y el 4096 es de 4096 bits que es para mejor seguridad, la misma quedó en /root/.ssh/

#Para copiar la clave de una VM a la otra, usé el comando “ssh-copy-id”

**Configurar Credenciales Globales en Jenkins:** 

#Entro nuevamente a Jenkins y voy a la opción “Credentials”: Selecciono el dominio “global” y presiono en “Add credentials”

#En la opción “kind” selecciono “SSH Username with private key, En Private Key, selecciono “Enter directly” y pego el contenido de la clave privada.

**Configurar un Agent en Jenkins**

#Para crear un agente en Jenkins, vamos a la opción “nodos - New node”  Le ponemos el nombre “VM2_Nodo” y lo seleccionamos del tipo agente permanente.

#Luego vamos a configurar el nodo, llenando los campos: "Nodo agente VM2"

#Método de ejecución: "Arrancar agentes remotos en máquinas UNIX vía SSH"

#Se configura el nombre de máquina, las credenciales y se ejecuta el agente configurado

**Configurar GitHub y Webhooks**

#Entramos a GitHub y creamos un nuevo repositorio, llamado en este caso “repojenkins”, copiamos la URL.

#Con el comando “git clone” lo copiamos al disco local

#Configurar el Webhook en GitHub : Dentro de GitHub vamos a la sección “Webhooks”, vamos a “Add webhook”

#En el campo “Payload URL” colocamos la URL de Jenkins seguido de “github-webhook” y en “Content type” la opción “application/json”, luego lo dejamos activo y configuramos como solamente los eventos “push”

**Crear el Pipeline de Jenkins**

#Crear un Jenkinsfile en el Repositorio: Creo un archivo Jenkinsfile, al mismo le creo contenido básico como primera prueba:"

<code>
pipeline {
    agent any
    stages {
  	stage('Checkout') {
	    steps {
		// Clona repo de git :)
		git 'https://github.com/pablonssss/repojenkins'
	    }
	}
    stage('Build') {
	    steps {
		// Hacemos de cuenta que vamos a compilar un proyecto...
		echo 'Compilandooo'
		// Aqui agregamos comandos para realizar una compilacion
	    }
   	}
    stage('Test') {
	    steps {
		// Ejecucion de pruebas
		echo 'Probandooo'
		// Aca comandos que realmente ejecuten pruebas
	    }
	}
    stage('Archive artefacts') {
	    steps {
		// Archivando artefacto que genero la compilacionnn
		echo 'Archivandooo'
	    }
   	}
    post {
	success {
	    echo 'Pipeline exitoso'
	}
	failure {
	    echo 'Pipeline fallo'
	}
    }
}
</code>





