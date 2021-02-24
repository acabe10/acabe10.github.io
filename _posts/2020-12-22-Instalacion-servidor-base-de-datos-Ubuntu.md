---
layout: post
title: Instalación y configuración servidor base de datos Ubuntu
tags: [all, OpenStack, Toboso, Servidor bbdd, Ubuntu]
---
# Introducción

Buenas, en este post vamos a instalar y configurar el servidor de base de datos para nuestro escenario en OpenStack, la instalación la haremos en `sancho`, que tiene un Ubuntu. También modificaremos nuestro servidor DNS instalado anteriormente para que reconozca la siguiente dirección `bd.cabezas.gonzalonazareno.org` como la base de datos y haremos una prueba desde quijote.

## Configuración en sancho

Instalamos:

~~~
sudo apt install mariadb-server
~~~

Hacemos la instalación segura:

~~~
sudo mysql_secure_installation
~~~

Entramos como root en la base de datos:

~~~
sudo mysql -u root -p
~~~

Y creamos al usuario con privilegios para acceder desde el exterior:

~~~
GRANT ALL PRIVILEGES ON *.* TO 'ale'@'%' IDENTIFIED BY 'ale' WITH GRANT OPTION;flush privileges;
~~~

Editamos el fichero:

~~~
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
~~~

Descomentamos las líneas(si están comentadas):

~~~
port                   = 3306
bind-address            = 127.0.0.1
skip-external-locking
~~~

Y modificamos la línea:

~~~
bind-address            = 0.0.0.0
~~~

Reiniciamos el servicio:

~~~
sudo systemctl restart mysql
~~~

## Configuración en freston

En los siguientes ficheros:

~~~
sudo nano /var/cache/bind/db.dmz.cabezas.gonzalonazareno.org
sudo nano /var/cache/bind/db.interna.cabezas.gonzalonazareno.org
~~~

Añadimos la siguiente línea:

~~~
bd              IN      CNAME   sancho
~~~

Reiniciamos el servicio:

~~~
sudo systemctl restart bind9
~~~

## Configuración en quijote

Debemos de instalar el paquete de `mariadb`, para ello:

~~~
sudo dnf install mariadb-server
~~~

Si quisieramos instalar una versión más actualizada, añadimos el repositorio de mariadb:

~~~
sudo nano /etc/yum.repos.d/mariadb-10.4.repo
~~~

Y añadimos:

~~~
[mariadb]
name = MariaDB para CentOS 7
baseurl = http://yum.mariadb.org/10.4/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
~~~

Activamos el servicio:

~~~
sudo systemctl start mariadb
~~~

## Comprobación

Desde quijote:

~~~
[centos@quijote ~]$ mysql -h bd.cabezas.gonzalonazareno.org -u ale -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 36
Server version: 10.3.25-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
~~~