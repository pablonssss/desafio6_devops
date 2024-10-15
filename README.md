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

#Agregamos el archivo al repositorio y hacemos commit: "git add Jenkinsfile" , luego "git commit -m "Jenkinsfile para pipeline de Jenkins"

#Luego push: "git push origin main"

**Ahora a la última parte del desafío: Crear un pipeline de Jenkins a través de un Jenkinsfile en un repositorio de GitHub para despliegue automático en un servidor Apache (VM2):**
**1.Verificar si Apache está instalado.**
**2.Configurar el servidor Apache.**
**3.Aplicar los cambios realizados en el repositorio de GitHub sobre Apache (index.html).**

#Primero verificamos que el Apache está instalado:
#Ejecutamos “sudo systemctl status apache2” para verificar, como no tenemos instalado aún apache, hacemos un “apt update” y “apt install apache2” para instalarlo

#Chequeamos el estado del servicio, luego de instalado: "sudo systemctl status apache2"

#Ahora creo el “Jenkinsfile” que en este caso hará lo siguiente según las etapas:
#Checkout: Clona el repositorio desde GitHub:
#Verifica si apache está instalado
#Configura el Apache para dejar el servicio iniciado
#“Deploy Apache” Copia el archivo index.html al directorio Apache de la VM2

<code>
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Clono el repo desde GitHub
                git 'https://github.com/pablonssss/repojenkins.git'
            }
        }

        stage('Verifico si el apache esta instalado') {
            steps {
                // Verifica si Apache está instalado en la VM2
                sshagent(['55c5edfe-cded-4a87-937b-b381846887a3']) {
                    sh 'ssh -o StrictHostKeyChecking=no pablo@10.0.2.17 "sudo systemctl status apache2 || sudo apt-get install apache2 -y"'
                }
            }
        }

        stage('Configure Apache') {
            steps {
                // Configura Apache en la VM2
                sshagent(['55c5edfe-cded-4a87-937b-b381846887a3']) {
                    sh 'ssh pablo@10.0.2.17 "sudo systemctl start apache2 && sudo systemctl enable apache2"'
                }
            }
        }

        stage('Deploy Apache') {
            steps {
                // Copia el archivo index.html al directorio de Apache en la VM2
                sshagent(['55c5edfe-cded-4a87-937b-b381846887a3']) {
                    sh 'scp index.html pablo@10.0.2.17:/var/www/html/index.html'
                }
            }
        }
    }

    post {
        success {
            echo 'Despliegue completado con éxito en Apache (VM2).'
        }
        failure {
            echo 'El pipeline ha fallado. Verifica los logs para más detalles.'
        }
    }
}
</code>

#Genero un archivo pablo.html lo creé para sustituir el index.html, lo renombro con el nombre index.html

#Subo el cambio a Git: “git add .” luego “git commit -m “nuevo Jenkinsfile y nuevo index.html” y al final el “push”

#Ahora vamos a crear el pipeline en Jenkins: Iniciamos sesión en Jenkins y vamos a “Nueva Tarea” 

#En la sección “Enter an item name”, le ponemos un nombre, en este caso Deploy Apache y debajo le seleccionamos que será de tipo “Pipeline”, rellenamos el formulario de la siguiente forma:

#En “SCM”, ponemos el valor “Git”, en “Repository URL” la URL del Git “repojenkins”, en “Branch Specifier”, colocamos “main” y en “Script Path” configuramos el campo como “Jenkinsfile”, nombre del archivo que Jenkins buscará en el repo para ejecutar el pipeline.

**Corrección de errores del pipeline anterior, solución de los mismos y resultado final del pipeline:**

#Al comienzo lanzamos la ejecución y uno de los errores que tenemos es el siguiente: “Couldn't find any revision to build. Verify the repository and branch configuration for this job”, 

#Investigando entiendo que Jenkins no está ejecutando correctamente la etapa de “checkout” de Git: No encuentro errores, pero lo que agrego es especificar puntualmente la rama donde está el Jenkinsfile: “git branch: 'main', url: 'https://github.com/pablonssss/repojenkins.git'” 

#Este error queda corregido, ahora en la siguiente ejecución encuentro otro error: “java.lang.NoSuchMethodError: No such DSL method 'sshagent' found among steps...”

#Este error se debe a que “sshagent” es un plugin que no tengo instalado en Jenkins por lo que procedo a instalarlo, en “Administrar Jenkins -> Plugins”

#Luego en la siguiente ejecución y con el error del comando “sshagent” solucionado, tengo el error que está mal la clave SSH, la corrijo.

#Posteriormente tengo el siguiente error: “sudo: a terminal is required to read the password; either use the -S option to read from standard input or configure an askpass helper” y “sudo: a password is required”

#Esto sucede porque el usuario con el que me conecto a la VM2, no tiene permisos de sudo en su equipo. Voy a la VM2 y edito el archivo /etc/sudoers, con el comando “sudo visudo” y agrego la última línea al archivo mencionado: “pablo ALL=(ALL) NOPASSWD: ALL”

#Corregido esto, el error ahora, ya se presenta en la última etapa (stage “Deploy en Apache”) da un error de permisos, porque falta la palabra “sudo” cuando voy a copiar el archivo index.html al directorio /var/www/html/index.html

#Corrijo este stage dejándolo de la siguiente forma:

#Primero copio el archivo index.html a /home/pablo/ y luego con sudo movemos el archivo a /var/www/html/

#Luego de corregir esto, la ejecución fue exitosa.

**El pipeline definitivo es el siguiente:**

<code>
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Clonar el repositorio desde GitHub
                git branch: 'main', url: 'https://github.com/pablonssss/repojenkins.git'
            }
        }

        stage('Verifico si Apache está instalado') {
            steps {
                // Verificar si Apache está instalado en VM2, instalar si es necesario
                sshagent(['1092453d-8fad-4168-9714-ba0d89b5773c']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no pablo@10.0.2.17 "
                        if ! systemctl is-active --quiet apache2; then
                            sudo apt-get update
                            sudo apt-get install apache2 -y
                        fi"
                    '''
                }
            }
        }

        stage('Configurar Apache') {
            steps {
                // Configurar y habilitar Apache en la VM2
                sshagent(['1092453d-8fad-4168-9714-ba0d89b5773c']) {
                    sh 'ssh pablo@10.0.2.17 "sudo systemctl is-active --quiet apache2 || sudo systemctl start apache2 && sudo systemctl enable apache2"'
                }
            }
        }

        stage('Deploy en Apache') {
            steps {
                // Primero copio el archivo a un directorio temporal y luego lo muevo con sudo
                sshagent(['1092453d-8fad-4168-9714-ba0d89b5773c']) {
                    sh '''scp -o StrictHostKeyChecking=no index.html pablo@10.0.2.17:/home/pablo/index.html
                    ssh pablo@10.0.2.17 "sudo mv /home/pablo/index.html /var/www/html/index.html"         '''
                }
            }
        }
    }

    post {
        success {
            // Mensaje en caso de éxito
            echo 'Despliegue completado con éxito en Apache (VM2).'
        }
        failure {
            // Mensaje en caso de fallo
            echo 'El pipeline ha fallado. Verifica los logs para más detalles.'
        }
    }
}
</code>
