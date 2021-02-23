---
layout: post
title: Despliegue OpenCMS Java
tags: [all, CMS, Java, Despliegue, OpenCMS]
---
# Introducción

Buenas, en este post vamos a instalar OpenCMS en una máquina de nuestro cloud privado y posteriormente vamos a hacer su despliegue a través de `apache2` y `tomcat` usando el protocolo AJP.

## Instalación requisitos

Actualizamos el sistema:

~~~
sudo apt update && sudo apt upgrade -y
~~~

Instalamos los paquetes necesarios para desplegar la aplicación:

~~~
sudo apt install tomcat9 zip mariadb-server openjdk-11-jre
~~~

## Configuración MariaDB

Entramos en mariadb como usuario root:

~~~
sudo mysql -u root -p
~~~

Cambiamos la contraseña:

~~~
> ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';
~~~

Y creamos base de datos y usuario que vamos a usar para opencms:

~~~
> CREATE DATABASE opencms;
> CREATE USER 'user_open'@'localhost' IDENTIFIED BY 'pass_open';
> GRANT ALL PRIVILEGES ON opencms.* TO 'user_open'@'localhost';
> FLUSH PRIVILEGES;
~~~

## Descarga de CMS y ubicación

Descargamos el cms de su página oficial y lo descomprimimos:

~~~
wget http://www.opencms.org/downloads/opencms/opencms-11.0.2.zip
unzip opencms-11.0.2.zip
~~~

Ahora tenemos que ubicar el fichero `opencms.war` que se ha descomprimido en la siguiente ruta:

~~~
sudo cp opencms.war /var/lib/tomcat9/webapps/
cd /var/lib/tomcat9/webapps/
sudo rm -r ROOT
sudo mv opencms.war ROOT.war
sudo systemctl restart tomcat9
~~~

## Instalación OpenCMS

Accedemos a la url para su instalación y aceptamos los términos:

![1](/assets/img/posts/cms-java/1.png)

Confirmamos que tenemos todos los prerequisitos:

![2](/assets/img/posts/cms-java/2.png)

Añadimos el usuario root y su contraseña root, y el usuario creado anteriormente, seleccionamos "Eliminar bd si existe":

![3](/assets/img/posts/cms-java/3.png)

Dejamos los módulos por defecto que queremos que se instalen:

![4](/assets/img/posts/cms-java/4.png)

Añadimos la dirección MAC de nuestro equipo, la URL que tendrá y su nombre:

![5](/assets/img/posts/cms-java/5.png)

Y comenzará la instalación:

![6](/assets/img/posts/cms-java/6.png)

Una vez finalizada, pulsamos en continuar:

![7](/assets/img/posts/cms-java/7.png)

![8](/assets/img/posts/cms-java/8.png)

Y nos aparecerá el siguiente error:

![9](/assets/img/posts/cms-java/9.png)

Esto es debido a que tenemos que eliminar el setup de la instalación que hemos realizado, para ello:

~~~
sudo nano /etc/tomcat9/catalina.properties
~~~

Y añadimos la siguiente línea:

~~~
wizard.enabled = false
~~~

Reiniciamos tomcat:

~~~
sudo systemctl restart tomcat9
~~~

Ahora ya podremos acceder a la página principal de Opencms:

![10](/assets/img/posts/cms-java/10.png)

## Apache y Tomcat9 (AJP)

Voy a poner la configuración realizada para usar el protocolo AJP, el cual no me ha funcionado.

Editamos el fichero:

~~~
sudo nano /etc/tomcat9/server.xml
~~~

Y descomentamos las siguientes líneas:

~~~
<Connector protocol="AJP/1.3"
               port="8009"
               redirectPort="8443" />
~~~

Creamos el archivo:

~~~
sudo nano /etc/apache2/workers.properties
~~~

Y añadimos lo siguiente:

~~~
# Definir un worker usando ajp13 
worker.list=worker1
# Definir las propiedades del worker (ajp13) 
worker.worker1.type=ajp13
worker.worker1.host=localhost
worker.worker1.port=8009
~~~

Le decimos a Apache2 que use el archivo worker que hemos creado:

~~~
nano /etc/apache2/mods-available/jk.conf
~~~

Y modificamos la siguiente línea:

~~~
JkWorkersFile /etc/apache2/workers.properties
~~~

Y en el virtualhost, editamos:

~~~
sudo nano /etc/apache2/sites-available/000-default.conf
~~~

Y añadimos lo siguiente dentro del virtualhost:

~~~
JkMount /opencms* worker1
~~~

Lo anterior no me ha funcionado, incluso he creado otra aplicación básica con un `helloworld` y tampoco, por ello explico a continuación como lo he realizado con proxy inverso http.

## Apache2 y Tomcat9 (proxy inverso http)

Primero comprobamos si las líneas que definen el conector HTTP están en su fichero correspondiente:

~~~
sudo nano /etc/tomcat9/server.xml
~~~

Y tenemos que tener las siguientes, si no, las añadimos:

~~~
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               URIEncoding="utf­8"
               redirectPort="8443" />
~~~

Activamos el módulo que permite hacer el proxy inverso y reiniciamos apache:

~~~
sudo a2enmod proxy_http
sudo systemctl restart apache2
~~~

Ahora editamos el virtualhost por defecto de apache:

~~~
sudo nano /etc/apache2/sites-enabled/000-default.conf
~~~

Y añadimos las siguientes líneas:

~~~
ProxyRequests Off
ProxyPreserveHost On

ProxyPass / http://localhost:8080/
ProxyPassReverse / http://localhost:8080/
~~~

Reiniciamos apache:

~~~
sudo systemctl restart apache2
~~~

Y ya podremos acceder mediante el navegador sin tener que introducir el puerto 8080:

![11](/assets/img/posts/cms-java/11.png)