
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


**IPs utilizadas en el reto 4:**

**IPs Privadas:**

- **Servicio Kubernetes:** 10.103.192.1
- **Servicio Load balancer:** 10.103.193.248

**IPs Públicas:**
- **Servicio Load balancer:** 34.171.91.170
- **DataBase:** 35.192.174.33


Cabe resaltar que para el desarrollo y cumplimiento de este reto no fue necesario instalar ningun otro paquete o libreria adicional ya que todos los recursos utilizados son nativos de GCP.


## 4. Guía paso a paso de instalación y configuración

Para este reto, se realizó el despliegue y configuración de cada uno de los elementos requeridos. Esta es una pequeña guía de todo el proceso realizado.

### Creación del clúster de Kubernetes.

Para esto se utilizó el siguiente tutorial (https://cloud.google.com/kubernetes-engine/docs/tutorials/persistent-disk).

Primero, es necesario activar Cloud Shell para poder ejecutar los comandos necesarios.

Después, es necesario habilitar las APIs de GKE y Clud SQL Admin:

```
gcloud services enable container.googleapis.com sqladmin.googleapis.com
```

En Cloud Shell, configura la región predeterminada para la CLI de Google Cloud:

```
gcloud config set compute/region us-central1-a
```

Establezca la PROJECT_IDvariable de entorno en el ID de su proyecto de Google Cloud:

```
export PROJECT_ID=reto-4-422404
```

Descargue los archivos de manifiesto de la aplicación desde el repositorio de GitHub:

```
git clone https://github.com/EsteTruji/kubernetes-manifests
```


Cambie al directorio con el kubernetes-manifests:

```
cd kubernetes-manifests
```

Establezca la ```WORKING_DIR``` variable de entorno:

```
WORKING_DIR=$(pwd)
```

### Creación del clúster de GKE.

Se debe un clúster de GKE para alojar el contenedor de tu aplicación de WordPress.

En Cloud Shell, cree un clúster de GKE llamado persistent-disk-tutorial (se puede colocar el nombre que se requiera):

```
CLUSTER_NAME=persistent-disk-tutorial
gcloud container clusters create $CLUSTER_NAME
```

Una vez creado, se debe conectar a su nuevo clúster:

```
gcloud container clusters get-credentials $CLUSTER_NAME --region us-central1-a
```

De esa forma, ya estará creado el clúster.

### Configuración NFS.

Para la configuración del NFS, se siguió paso a paso el siguiente tutorial (https://cloud.google.com/filestore/docs/csi-driver?hl=es-419#deployment).

Antes de comenzar, es necesario realizar las siguientes tareas:

- Habilita la API de Cloud Filestore y la API de Google Kubernetes Engine.

Si se desea usar Google Cloud CLI para esta tarea, instale y, luego, initialize gcloud CLI. Si ya instaló gcloud CLI, ejecute gcloud components update para obtener la versión más reciente.

#### Habilita el controlador de CSI de Filestore en un clúster existente

Para habilitar el controlador de CSI de Filestore en clústeres existentes, usa Google Cloud CLI o la consola de Google Cloud.

- En la consola ejecute el siguiente comando:

```
  gcloud container clusters update CLUSTER_NAME \
   --update-addons=GcpFilestoreCsiDriver=ENABLED
```
Donde se debe reemplazar **CLUSTER_NAME** por el nombre del clúster.

#### Crea un PersistentVolume y una PersistentVolumeClaim para acceder a la instancia.

1. Crea un archivo de manifiesto como el que se muestra en el siguiente ejemplo y asígnale el nombre ```nfs-pv-pvc.yaml```:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  storageClassName: ""
  capacity:
    storage: 1Ti
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem
  csi:
    driver: filestore.csi.storage.gke.io
    volumeHandle: "modeInstance/us-central1-a  /nfs-server/nfs_share1"
    volumeAttributes:
      ip: 10.34.43.66
      volume: nfs_share1
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: podpvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ""
  volumeName: nfs-pv
  resources:
    requests:
      storage: 1Ti
```

- Posteriormente, ejecute el siguiente comando para crear los recursos de ```PersistentVolume``` y ```PersistentVolumeClaim```:

```
kubectl apply -f nfs-pv-pvc.yaml
```

De esta forma, ya se tiene creada y configurada la instancia de NFS usando el servicio de FileStore de GCP.
Es posible visualizarlo al ingresar al clúster correspondiente, y dirigirse a la sección **Almacenamiento**, en la parte de abajo.





### Configuración Base de Datos.

En Cloud Shell, cree una instancia denominada ```mysql-wordpress-instance```:

```
INSTANCE_NAME=mysql-wordpress-instance
gcloud sql instances create $INSTANCE_NAME
```

Agregue el nombre de la conexión de la instancia como una variable de entorno:

```
export INSTANCE_CONNECTION_NAME=$(gcloud sql instances describe $INSTANCE_NAME \
    --format='value(connectionName)')
```

Cree una base de datos para WordPress para almacenar sus datos:

```
gcloud sql databases create wordpress --instance $INSTANCE_NAME
```

Cree un usuario de base de datos llamado ```wordpress``` y una contraseña para que WordPress se autentique en la instancia:

```
CLOUD_SQL_PASSWORD=$(openssl rand -base64 18)
gcloud sql users create wordpress --host=% --instance $INSTANCE_NAME \
    --password $CLOUD_SQL_PASSWORD
```

Así, quedará lista la instancia de la base de datos.


### Configuración WordPress.

#### Configuración inicial.

Antes de poder implementar WordPress, se debe crear una cuenta de servicio. Proceda a crear un secreto de Kubernetes para guardar las credenciales de la cuenta de servicio y otro secreto para guardar las credenciales de la base de datos.

Para permitir que su aplicación de WordPress acceda a la instancia de MySQL a través de un proxy de Cloud SQL, cree una cuenta de servicio:

```
SA_NAME=cloudsql-proxy
gcloud iam service-accounts create $SA_NAME --display-name $SA_NAME
```

Agregue la dirección de correo electrónico de la cuenta de servicio como variable de entorno:

```
SA_EMAIL=$(gcloud iam service-accounts list \
    --filter=displayName:$SA_NAME \
    --format='value(email)')
```

Agregue el ```cloudsql.clientrol``` a su cuenta de servicio:

```
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --role roles/cloudsql.client \
    --member serviceAccount:$SA_EMAIL
```

Cree una clave para la cuenta de servicio:

```
gcloud iam service-accounts keys create $WORKING_DIR/key.json \
    --iam-account $SA_EMAIL
```

Cree un secreto de Kubernetes para las credenciales de MySQL:

```
kubectl create secret generic cloudsql-db-credentials \
    --from-literal username=wordpress \
    --from-literal password=$CLOUD_SQL_PASSWORD
```

Cree un secreto de Kubernetes para las credenciales de la cuenta de servicio:

```
kubectl create secret generic cloudsql-instance-credentials \
    --from-file $WORKING_DIR/key.json
```

#### Implementación.

El siguiente paso es implementar su contenedor de WordPress en el clúster de GKE.

El ```wordpress_cloudsql.yaml``` archivo de manifiesto describe una implementación que crea un único Pod que ejecuta un contenedor con una instancia de WordPress. Este contenedor lee la ```WORDPRESS_DB_PASSWORD``` variable de entorno que contiene el ```cloudsql-db-credentials``` secreto que creó.

Este archivo de manifiesto también configura el contenedor de WordPress para comunicarse con MySQL a través del proxy de Cloud SQL que se ejecuta en el contenedor sidecar . El valor de la dirección del host se establece en la ```WORDPRESS_DB_HOST``` variable de entorno.

Prepare el archivo reemplazando la ```INSTANCE_CONNECTION_NAME``` variable de entorno:

```
cat $WORKING_DIR/wordpress_cloudsql.yaml.template | envsubst > \
    $WORKING_DIR/wordpress_cloudsql.yaml
```

Implemente el ```wordpress_cloudsql.yaml``` archivo de manifiesto:

```
kubectl create -f $WORKING_DIR/wordpress_cloudsql.yaml
```

Mire la implementación para ver el cambio de estado a ```running```:

```
kubectl get pod -l app=wordpress --watch
```

Cuando la salida muestra un estado de ```Running```, puede pasar al siguiente paso.


#### Exposición del servicio.

Crear un servicio de ```type:LoadBalancer```:

```
kubectl create -f $WORKING_DIR/wordpress-service.yaml
```

Observe la implementación y espere a que el servicio tenga asignada una dirección IP externa:

```
kubectl get svc -l app=wordpress --watch
```

Cuando el resultado muestre una dirección IP externa, puede continuar con el siguiente paso. 

Tome nota del ```EXTERNAL_IP``` campo de dirección para utilizarlo más adelante.


#### Configuración del blog/página.

En su navegador, vaya a la siguiente URL, reemplazando ```external-ip-address``` con la ```EXTERNAL_IP``` del servicio que expone tu instancia de WordPress:

```
http://external-ip-address
```

Siga después los pasos a continuación:

- En la página de instalación de WordPress , seleccione un idioma y luego haga clic en Continuar .

- Complete la página Información necesaria y luego haga clic en Instalar WordPress .

- Haga clic en Iniciar sesión.

- Ingrese el nombre de usuario y contraseña que creó anteriormente.

Una vez todo está configurado, es posible ejecutar el siguiente comando para poder visualizar todos los pods, nodos, servicio y deployments creados en el cluster de Kubernetes:

```
kubectl get all
```

Mostrará algo así entonces:

![image](https://github.com/EsteTruji/st0263-reto-4/assets/82886890/1e55fd24-3167-46c0-b172-77a06480cc2f)


#### Configuración dominio.

Para esto, es necesario dirigirse al panel de administración del proveedor de DNS, para nuestro caso es GoDaddy. Allí nos dirigimos a la sección de registros DNS.

Ahora procedemos a agregar el dominio (con el valor de la IP externa previamente obtenida) el subdominio en el DNS de igual forma y finalmente el registro ```CNAME```. Dentro de la configuración de registros del proveedor, se vería de la siguiente manera: 


```
A        @        IP expuesta del balanceador
A        reto4    IP expuesta del balanceador
CNAME    www      reto4.toysnt.shop
```

- Ahora, al acceder al enlace **http://reto4.toysnt.shop** podrá visualizar la página web desplegada y funcionando sin problema (tenga presente que para el momento de la revisión puede que las instancias que alojan todos los recursos y procesos previamente enunciados no estén en ejecución, y por lo tanto no esté disponible la página web).

![image](https://github.com/EsteTruji/st0263-reto-4/assets/82886890/24841215-8aa4-46a6-825e-6721c4a7ae77)


## Referencias

[Deploy Wordpress on Kubernetes in GCP the easy way!](https://www.youtube.com/watch?v=R7FI05Q-nkg)

[Deploy WordPress on GKE with Persistent Disk and Cloud SQL](https://cloud.google.com/kubernetes-engine/docs/tutorials/persistent-disk?hl=es-419)

[Accede a las instancias de Filestore con el controlador de CSI de Filestore](https://cloud.google.com/filestore/docs/csi-driver?hl=es-419#deployment)
