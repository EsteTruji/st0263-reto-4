
# Info de la materia: ST0263 - Tópicos Especiales en Telemática

### Estudiantes:
  - Esteban Trujillo Carmona, etrujilloc@eafit.edu.co
  - Viviana Hoyos Sierra, vhoyoss@eafit.edu.co
  - Damian Duque López, daduquel@eafit.edu.co
  - Valentina Morales Villada, vmoralesv@eafit.edu.co

### Profesor: Edwin Montoya, emontoya@eafit.edu.co

# Reto 4 

# Video sustentación

- Enlace: **...**

## 1. Breve descripción de la actividad
Se desplegó un CMS wordpress (app monolítica) en una arquitectura distribuida haciendo uso de tecnologías como Kubernets, proveedores de dominio como GoDaddy y servicios de Google cloud.

En este reto utilizamos los servicios de balanceador de cargas, Filestore y Cloud SQL nativos de google para satisfacer las necesidades de nuestra aplicación. utilizando un servicio administrado de NFS en la nube a partir de la API de Filestore para disponer del sistema de archivos. se desplegaron (PENDIENTE POR CONFIRMAR Y ANOTAR) maquinas virtuales en google cloud.



### 1.1. Que aspectos cumplió o desarrolló de la actividad propuesta por el profesor (requerimientos funcionales y no funcionales)

**NO FUNCIONALES**

-	**DISPONIBILIDAD:**
    -  Despliegue de  aplicación wordpress con varios pods utilizando Kubernetes.
    -  Implementar un balanceador de cargas basado en los servicios nativos de Google que reciba el tráfico web http de Internet con múltiples instancias de procesamiento.
   

**FUNCIONALES**
-	Almacenamiento en la capa de datos haciendo uso del servicio de bases de datos mysql ofrecido por Google.
- Almacenamiento de archivos implementado haciendo uso del servicio de NFS ofrecido por Google.
- Conexion de servicios de wordpress con el servicio NFS.



### 1.2. Que aspectos NO cumplió o desarrolló de la actividad propuesta por el profesor (requerimientos funcionales y no funcionales)

No logro obtener el certificado SSL para implementar https dentro de la aplicación.

## 2. Información general de diseño de alto nivel, arquitectura, patrones, mejores prácticas utilizadas.

Se implementó para este proyecto la siguiente arquitectura basada en la propuesta del profesor.

[Diagrama](https://drive.google.com/file/d/11rlnW3-tqprr7dtQRbYrXvi0q4SP7CP8/view?usp=sharing)

![Reto3-TT drawio](https://github.com/EsteTruji/st0263-reto-3/assets/81714113/2c14edb4-a5c9-40be-aeb5-a9f91cfdc248)

## 3. Descripción del ambiente de ejecución lenguaje de programación, librerias, paquetes, etc, con sus numeros de versiones.


IPs utilizadas en el reto 4:

IPs Privadas:

- WordPress: 
- WordPress2:
- Load balancer:
- DataBase: 
- NFS: 

IPs Públicas:
- Load balancer:


Cabe resaltar que para el desarrollo y cumplimiento de este reto no fue necesario instalar ningun otro paquete o libreria adicional ya que todos los recursos utilizados son nativos de GCP.


## 4. Guía paso a paso de instalación y configuración

Para este proyecto, se realizó el despliegue de varias instancias según lo solicitado en la descripción. Esta es una pequeña guía del proceso de instalación y configuración de estas instancias.

- Se crea la VPC donde van a estar almacenadas las instancias que crearemos. Esta está compuesta de dos subredes, una pública y otra privada.

![screenshot 2024-04-20 (1)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/7eed552d-e3c2-4d62-90dd-553dfa44900a)


![screenshot 2024-04-16 (1)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/32d1a6fd-c3f9-452d-a998-247eab15263b)


![screenshot 2024-04-16 (2)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/5c4979cf-aea1-4bda-9837-f9f4e31fd479)


- Se crean 3 instancias dentro de esta subred privada, cada una para los siguientes servicios Database, Wordpress, NFS. Todas estas con ips privadas.

![screenshot 2024-04-16 (7)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/14adb39b-56dd-44bf-9b66-ca42b792de4f)


![screenshot 2024-04-16 (8)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/83e7de4a-8fd9-4114-9821-dcfbb21636b9)


![screenshot 2024-04-16 (9)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/05e73e45-d973-4c66-8023-46a7696f4744)


- Dentro de la subred pública de la vpc se crean 2 instancias.
    - La instancia del Load Balancer y se le asocia una ip pública elástica.

    ![screenshot 2024-04-16 (10)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/c4a54706-e013-420c-a415-b420371d3ef9)


    - Se crea la instancia de Bastion Host y se le asocia una ip pública.

- Así ya tenemos las cinco instancias dentro de la VPC que creamos inicialmente.

![screenshot 2024-04-16 (11)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/5f18a288-8880-4a66-8960-2e8214ade7ca)


![screenshot 2024-04-16 (12)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/d29c8fc6-d030-4be8-9258-076b68f72617)



- Ahora procedemos a agregar el dominio (con el valor de la IP elástica del LB)  el subdominio en el DNS de igual forma y finalmente el registro CNAME. Dentro de la configuración de registros del proveedor, se vería de la siguiente manera: 

![image](https://github.com/EsteTruji/st0263-reto-4/assets/94024545/cdd851f3-6b26-4a51-b846-68b0604e49f2)




```
A        @        IP elástica del balanceador
A        reto3    IP elástica del balanceador
CNAME    www      reto3.toysnt.shop
```

- Modificamos las reglas del security group de la instancia de Database para asociarla con una base de datos MySQL.

![screenshot 2024-04-16 (14)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/7f2eeafa-80fa-4d80-a454-489bd2baf83d)


![screenshot 2024-04-16 (15)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/76d2d17d-fe38-4b1e-b9d2-e4a8302fbb44)


![screenshot 2024-04-16 (16)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/19861801-9158-402b-816c-c20abff10ed9)



- Ahora abrimos la consola de Ubuntu 22.04 localmente, y accedemos mediante la clave .pem, a la instancia remota del Bastion Host.

![screenshot 2024-04-16 (17)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/39397958-6d58-423d-b80f-a8f389c7b35e)


![screenshot 2024-04-16 (19)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/1b0560de-ac44-4171-882f-085a227c861f)



- Ahora estando dentro de la instancia del Bastion Host, accedemos a la instancia de Database mediante la clave .pem
  
  * Debemos ejecutar el siguiente comando para darle permisos al usuario para escribir y leer el archivo .pem

  ```
  chmod 400 "reto-3.pem"                    
  ```

  * Editamos el archivo reto-3.pem con la clave que descargamos.

![screenshot 2024-04-16 (20)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/f722fb5d-2d4b-469b-8871-22561400ea10)


![screenshot 2024-04-16 (21)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/15f24aa7-543b-4211-a562-2017697c76ea)




- Instalamos Docker en la instancia de Database y editamos el archivo de docker compose para declarar las variables de entorno que nos permitirán conectarnos a la base de datos MySQL.

![screenshot 2024-04-16 (23)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/a4807018-4120-4310-8bf3-08c3530685de)


![screenshot 2024-04-16 (22)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/4d24deb0-963a-4a71-9d4a-74f4cc63eddc)



- Y finalmente ejecutamos docker compose up para correr la instancia de base de datos.

![screenshot 2024-04-16 (24)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/220a66bb-5738-45d0-b883-bc8558b3a3fb)


![screenshot 2024-04-16 (25)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/48ec14ab-fa93-4017-8272-8ac6d818d276)

### Configuración NFS.

- Ahora vamos a configurar la instancia de NFS (Network File System), para esto debemos escoger primero dentro del security group el puerto 2049 NFS y adicionalmente debemos configurar un *firewall* ya que como en el servidor NFS se estarán almacenando archivos, debemos incluir una capa extra de seguridad. Por esto lo que debemos habilitar también el puerto 22 correspondiente al SSH.

- Para la instalación del NFS server, se deben utilizar los siguientes comandos:

a. Para instalar el paquete ```nfs-kernel.server```, así:
```
sudo apt update
sudo apt install nfs-kernel-server -y
```

- Al finalizar esta instalación, tendremos creado el directorio mnt que es donde se guardarán los archivos de las instancias de Wordpress de la siguiente manera:
a. Se crea el directorio de Wordpress:
```
sudo mkdir -p /mnt/wordpress
```

b. Se modifica el propietario del directorio:

```
sudo chown nobody:nogroup /mnt/wordpress
```

c. Se modifican los permisos del dorectorio creado:

```
 sudo chmod 777 /mnt/wordpress
```

d. Se cambia la configuración de los directorios a exportar a través de NFS:

```
sudo nano /etc/exports

/mnt/wordpress *(rw,sync,no_subtree_check)
```

e. Se activa el firewall_

```
sudo ufw enable
```

f. Se agrega la regla para permitir conexión por NFS y SSH:

```
sudo ufw allow nfs

sudo ufw allow ssh
```

### Configuración Wordpress.

- Ahora procedemos con la configuración de la instancia de Wordpress, que luego más adelante estaremos duplicando para asignar ambas instancias al load balancer y que este pueda distribuir las peticiones entre las dos instancias de Wordpress.

- Configuramos el security group de la instancia con el puerto 80 correspondiente a HTTP.

- Instalamos docker:

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

```
# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

- Luego pasamos a configurar el NFS Client ya que esta instancia es la que actuará como cliente del NFS. Esto lo podemos hacer con los siguientes comandos:

a. Se instala ```nfs-common```:

```
sudo apt update
sudo apt install nfs-common -y
```

b. Se crea el directorio de montaje:

```
 sudo mkdir -p /mnt/wordpress
```

c. Se monta el directorio:

```
sudo mount <IP-PRIVADA-NFS>:/mnt/wordpress /mnt/wordpress
```

- Una vez realizado este paso, vamos a editar el docker-compose.yml y le ponemos la información necesaria para poder conectarnos con la base de datos (el archivo docker compose correspondiente se encuentra en la carpeta Wordpress de este repositorio).

- Ahora podemos ejecutar el comando docker compose up para dejar la instancia de Wordpress corriendo correctamente y que podamos editar la página desde la página de administración de contenido.

```
sudo docker compose up -d
```

![screenshot 2024-04-20 (6)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/b7de5512-d1b1-4f4f-a1f9-015746c13da0)


![screenshot 2024-04-20 (7)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/8cf628c1-65a8-4010-b2ee-2f1071f2dff4)


### Configuración Load Balancer.

- Finalmente, vamos a configurar la instancia de load balancer, que como ya habíamos mencionado, nos permitirá distribuir las cargas de las peticiones entre las dos instancias de Wordpress que tenemos. Para este proyecto utilizaremos como servidor a NGINX.

- Para esto primero escogemos en el security group agregramos los puertos 80 y 443 para el acceso a HTTP y HTTPS respectivamente.

- Ahora vamos a instalar docker como en las demás instancias que lo requieren.

- Instalamos docker:

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

```
# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

- Además se deben instalar los paquetes letsencrypt, certbot y nginx, de la siguiente manera:

```
sudo apt update
sudo add-apt-repository ppa:certbot/certbot
sudo apt install letsencrypt -y
sudo apt install nginx -y
```

- Se procede a ejecutar el comando de las librerías de certbot y letsencrypt. Esto genera el nombre (acme-challenge) y el valor es una cadena de caracteres encriptada, estos se ingresan en el registro TXT en el proveedor. Antes de continuar, hay que esperar a que se realice la propagación en los servidores del provedor, para validar la disponibilidad del registro se puede ingresar a el "Google Admin Tool Box".

```
sudo mkdir -p /var/www/letsencrypt
sudo certbot --server https://acme-v02.api.letsencrypt.org/directory -d *.toysnt.shop --manual --preferred-challenges dns-01 certonly
```
_Fíjese que se está utilizando la wild card (*) para obtener el certificado SSL._


![screenshot 2024-04-20 (3)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/1840646d-9455-4a2f-b666-71b825572764)


![screenshot 2024-04-20 (5)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/bed13529-482b-4008-97c0-6a2cf67bb46c)



- Cuando en el toolbox se ingrese el nombre del registro y lo encuentre (devolviendo el valor correspondiente) este ya se encontrará disponible y procedemos con la operación en consola (con la tecla enter).
- Ahora, se debe crear un directorio de nombre **loadbalancer** y dentro de este otro de nombre **ssl** en donde se almacenarán los archivos asociados al certificado y la llave para https, de esta manera:

```
mkdir loadbalancer
mkdir loadbalancer/ssl
```

- Después, se copian los certificados creados:

```
sudo su
cp /etc/letsencrypt/live/toysnt.shop/* /home/ubuntu/loadbalancer/ssl
```

- Así, al ingresar a la carpeta ssl podrá visualizar las claves y certificados recién copiados.

```
.
└── loadbalancer
    └── ssl
        ├── cert.pem
        └── chain.pem
        └── fullchain.pem
        └── privkey.pem

```

- Salimos del directorio ssl, y en la carpet raíz se procede a crear el archivo de configuración para SSL con el siguiente comando:

```
nano loadbalancer/ssl.conf
```
_El contenido que deberá ir dentro de este archivo se encuentra en este repositorio en la carpeta Load-balancer. Proceda a copiarlo y pegarlo entonces._

- Qudará entonces así el direcotorio loadbalancer, por el momento:

```
└───loadbalancer
    │   ssl.conf
    │
    └───ssl
```

En esta configuración se especifican valores para el acceso a los archivos generados con anterioridad.

![screenshot 2024-04-20 (8)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/a7b4cc8a-3c4e-4bfb-9849-50ac8fcf2055)


![screenshot 2024-04-20 (10)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/79ebb9c4-88e7-4c86-8122-798d3eecc4af)


- Ahora vamos a crear el archivo de configuración nginx.conf para el load balancer, así:

```
nano loadbalancer/nginx.conf
```
_El contenido que deberá ir dentro de este archivo se encuentra en este repositorio en la carpeta Load-balancer. Proceda a copiarlo y pegarlo entonces._


- Posteriormente, ya dentro de este archivo se especifica la IP de la instancia de Wordpress que tenemos hasta el momento. Se debe ubicar en el campo indicado a continuación como ```<wordpress_ip_1>```:

```
upstream backend {
      server <wordpress_ip_1>;
   }
```

- Ahora, la carpeta loadbalancer estará de la siguiente manera:
  
```
└───loadbalancer
    │   nginx.conf
    │   ssl.conf
    │
    └──ssl
```


![ss12 copia](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/2897b03e-bff1-467f-b3a6-97ff3624320a)



- Después, creamos el archivo docker-compose.yml en donde insertaremos la estructura base para nginx necesaria.

```
nano loadbalancer/docker-compose.yml
```
_El contenido que deberá ir dentro de este archivo se encuentra en este repositorio en la carpeta Load-balancer. Proceda a copiarlo y pegarlo entonces._

- Luego, se debe detener cualquier proceso o ejecución activa de NGINX en la instancia para evitar conflictos:

```
ps ax | grep nginx
netstat -an | grep 80

sudo systemctl disable nginx
sudo systemctl stop nginx
exit
```

- Luego ejecutamos el comando de docker compose up para correr la instancia del load balancer nginx y que funcione correctamente.

```
cd loadbalancer
docker compose up -d
```

### Configuración Wordpress 2.

- Ahora podemos clonar la instancia que tenemos de Wordpress y nombrarla 'wordpress-2', la cual tendrá la misma configuración que la instancia 'wordpress'.
- 
- Se actualiza entonces el archivo nginx.conf con la ip privada de la nueva instancia 'wordpress-2'.
  
![screenshot 2024-04-20 (12)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/57ebceef-d34c-4803-9836-07be325ddc16)


- Finalmente se recarga la página y así quedarían las instancias dentro de la VPC.

![screenshot 2024-04-20 (2)](https://github.com/EsteTruji/st0263-reto-3/assets/83479274/b3c498ed-9984-4280-91cd-ccbac816592f)

- Ahora, al acceder al enlace **https://reto3.toysnt.shop** podrá visualizar la página web desplegada y funcionando sin problema (tenga presente que para el momento de la revisión puede que las instancias que alojan todos los recursos y procesos previamente enunciados no estén en ejecución, y por lo tanto no esté disponible la página web).


## Referencias

[Deploy Wordpress on Kubernetes in GCP the easy way!](https://www.youtube.com/watch?v=R7FI05Q-nkg)

[Deploy WordPress on GKE with Persistent Disk and Cloud SQL](https://cloud.google.com/kubernetes-engine/docs/tutorials/persistent-disk?hl=es-419)

[Accede a las instancias de Filestore con el controlador de CSI de Filestore](https://cloud.google.com/filestore/docs/csi-driver?hl=es-419#deployment)
