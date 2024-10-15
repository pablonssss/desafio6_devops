# desafio6_devops
Trabajo desafio 6  - Jenkins y Apache

**Instalar Jenkins en VM1** 

#Chequeo si tengo Java instalado

java --version

#El paso siguiente es agregar la clave GPG para garantizar la seguridad del paquete de Jenkins a instalar, para ello ejecutamos los comandos: 
"curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null"

#y añadimos el repositorio
"signed-by=/usr/share/keyrings/jenkins-keyring.asc" esto le indica al sistema que los paquetes de este repositorio deben ser verificados utilizando la clave GPG que se guardó anteriormente. 
