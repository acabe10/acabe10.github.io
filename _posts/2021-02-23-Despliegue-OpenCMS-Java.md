---
layout: post
title: Despliegue OpenCMS Java
tags: [all, CMS, Java, Despliegue, OpenCMS]
---
# Introducción

Buenas, en este post vamos a instalar OpenCMS en una máquina de nuestro cloud privado y posteriormente vamos a hacer su despliegue a través de `apache2` y `tomcat` usando un proxy inverso.

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