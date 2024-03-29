## WorkShop - Red Hat OpenShift Container Platform

## Crear una imagen de contenedor del servidor HTTP Apache con un Containerfile e implementarla en un clúster de OpenShift, se pueden seguir los siguientes pasos:

## Cree un nuevo proyecto para la aplicación. Coloque como prefijo del nombre del proyecto apache-prueba

oc new-project apache-prueba

## Crear un archivo llamado Dockerfile o Containerfile con el siguiente contenido
<pre>
FROM ubi8/ubi:8.3
MAINTAINER Yesid Cardenas ycardena@redhat.com
LABEL description="A custom Apache container based on UBI 8"
RUN yum install -y httpd && \
 yum clean all
RUN echo "Hello from Containerfile" > /var/www/html/index.html
EXPOSE 80
CMD ["httpd", "-D", "FOREGROUND"]
</pre>

## Compile la imagen secundaria del servidor HTTP Apache
podman build --layers=false  -t apache-prueba .

## Vea las imágenes:
podman images

## Etiquete y suba la imagen Quay.io.
podman tag apache-prueba quay.io/usuarioquay/apache-prueba 

podman login quay.io -u usuarioquay

podman push --format v2s1 quay.io/usuarioquay/apache-prueba

## Inicie sesión en Quay.io [https://quay.io] y haga que la nueva imagen sea pública.

## Implemente la imagen secundaria del servidor HTTP Apache:
oc new-app --name apache quay.io/usuarioquay/apache-prueba

## Verifique que el pod de la aplicación 
oc get pods

## Inspeccione los registros del contenedor para ver el motivo por el que el pod no logró iniciarse:
oc logs pod-name

## NOTA:
Debido a que OpenShift ejecuta contenedores con un ID de usuario aleatorio, los puertos inferiores a 1024 cuentan con privilegios y solo se pueden ejecutar como root.
El ID de usuario aleatorio que usa OpenShift para ejecutar el contenedor no tiene permisos de lectura ni de escritura de archivos de registro en /var/log/httpd (la ubicación predeterminada del archivo de registro para el servidor HTTP Apache en RHEL 7).

## Cree una ruta OpenShift para exponer la aplicación a acceso externo:
oc expose --port='8080' svc/nameservice

## Obtenga la URL de enrutamiento con el comando oc get route:
oc get route

## Pruebe la aplicación con la URL de ruta que obtuvo en el paso anterior:
curl http://route.apps
