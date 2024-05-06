
# Info de la materia: ST0263 - Tópicos Especiales en Telemática

### Estudiantes:
  - Esteban Trujillo Carmona, etrujilloc@eafit.edu.co
  - Viviana Hoyos Sierra, vhoyoss@eafit.edu.co
  - Damian Duque López, daduquel@eafit.edu.co
  - Valentina Morales Villada, vmoralesv@eafit.edu.co

### Profesor: Edwin Montoya, emontoya@eafit.edu.co

# Reto 4 

# Video sustentación

[![Video](https://img.youtube.com/vi/6Vu73vi8l-g/0.jpg)](https://www.youtube.com/watch?v=6Vu73vi8l-g)

## 1. Breve descripción de la actividad
Se desplegó un CMS wordpress (app monolítica) en una arquitectura distribuida haciendo uso de tecnologías como Kubernets, proveedores de dominio como GoDaddy y servicios de Google Cloud Platform.

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

![Reto4](https://github.com/EsteTruji/st0263-reto-4/assets/83479274/c6d99571-ebb8-43b5-baf1-24e7ce01638d)


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

Para este reto, se realizó el despliegue y configuración de cada uno de los elementos requeridos. Esta es una pequeña guía de todo el proceso realizado.


### Configuración NFS.

Para la configuración del NFS, 


- Ahora procedemos a agregar el dominio (con el valor de la IP elástica del LB)  el subdominio en el DNS de igual forma y finalmente el registro CNAME. Dentro de la configuración de registros del proveedor, se vería de la siguiente manera: 


```
A        @        IP elástica del balanceador
A        reto4    IP elástica del balanceador
CNAME    www      reto4.toysnt.shop
```










### Configuración Wordpress.








- Ahora, al acceder al enlace **https://reto4.toysnt.shop** podrá visualizar la página web desplegada y funcionando sin problema (tenga presente que para el momento de la revisión puede que las instancias que alojan todos los recursos y procesos previamente enunciados no estén en ejecución, y por lo tanto no esté disponible la página web).


## Referencias

[Deploy Wordpress on Kubernetes in GCP the easy way!](https://www.youtube.com/watch?v=R7FI05Q-nkg)

[Deploy WordPress on GKE with Persistent Disk and Cloud SQL](https://cloud.google.com/kubernetes-engine/docs/tutorials/persistent-disk?hl=es-419)

[Accede a las instancias de Filestore con el controlador de CSI de Filestore](https://cloud.google.com/filestore/docs/csi-driver?hl=es-419#deployment)
