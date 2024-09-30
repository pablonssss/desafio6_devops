# desafio6_devops
Trabajo desafio 6  - Jenkins y Apache

Vamos con la instalación de Jenkins, para ello se precisa Java instalado. Ya contamos con Java en esa VM desde un desafío anterior:
(img1)
El paso siguiente es agregar la clave GPG para garantizar la seguridad del paquete de Jenkins a instalar, para ello ejecutamos los comandos:
"curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null"
(img2)
y añadimos el repositorio 

"signed-by=/usr/share/keyrings/jenkins-keyring.asc" esto le indica al sistema que los paquetes de este repositorio deben ser verificados utilizando la clave GPG que se guardó anteriormente.
(img4) 
(img3)

Ejecuto para instalar Jenkins pero el sistema no encuentra el paquete Jenkins para instalar.
(img5)

Entonces, primero verifico que el contenido del archivo jenkins.list esté correcto y tiro un ping a "pkg.jenkins.io" para confirmar que tengo conectividad:
(img6)

Procedo mejor a hacer la instalación manual:"wget https://pkg.jenkins.io/debian-stable/binary/jenkins_2.414.2_all.deb"
(img7)
Luego de descargado lo instalo con dpkg:
(img8)
Al tener errores con las dependencias ejecuto "apt-get install -f" 

