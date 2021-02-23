---
layout: post
title: Instalación MariaDB, phpMyAdmin y acceso remoto
tags: [all, MariaDB, phpMyAdmin, BBDD]
---
# Introducción

Buenas, en este post vamos a instalar y configurar MariaDB para su acceso remoto.

## Instalación de MariaDB

Vamos a instalar MariaDB para posteriormente acceder a sus datos a través de una aplicación web. Para ello:

~~~
sudo apt install mariadb-server
~~~

Hacemos la instalación segura:

~~~
sudo mysql_secure_installation
~~~

## Creación de usuario y base de datos poblada

Vamos a crear el usuario que posteriormente accederemos remotamente mediante la aplicación web, para ello accedemos a mysql:

~~~
sudo mysql -u root -p
~~~

Y creamos el usuario:

~~~
GRANT ALL PRIVILEGES ON *.* TO 'externo'@'%' IDENTIFIED BY 'externo' WITH GRANT OPTION;flush privileges;
~~~

Para crear la base de datos e introducir datos salimos primero de `root` con `exit` y:

~~~
mysql -u externo -p

MariaDB [(none)]> create database prueba;
Query OK, 1 row affected (0.001 sec)

MariaDB [(none)]> use prueba
Database changed
MariaDB [prueba]> 
~~~

Y empezamos a introducir datos:

~~~
CREATE TABLE Provincias
(
	Codigo VARCHAR(10),
	Nombre VARCHAR(20),
	CONSTRAINT pk_provincias PRIMARY KEY(Codigo)
);

INSERT INTO Provincias (Codigo,Nombre) VALUES ('0','Sevilla');
INSERT INTO Provincias (Codigo,Nombre) VALUES ('1','Huelva');
INSERT INTO Provincias (Codigo,Nombre) VALUES ('2','Granada');
INSERT INTO Provincias (Codigo,Nombre) VALUES ('3','Cadiz');
INSERT INTO Provincias (Codigo,Nombre) VALUES ('4','Malaga');
INSERT INTO Provincias (Codigo,Nombre) VALUES ('5','Jaen');
INSERT INTO Provincias (Codigo,Nombre) VALUES ('6','Almeria');
INSERT INTO Provincias (Codigo,Nombre) VALUES ('7','Murcia');

CREATE TABLE Poblaciones
(
	Codigo VARCHAR(10),
	Nombre VARCHAR(20),
	CodigoProvincia VARCHAR(10),
	CONSTRAINT pk_poblaciones PRIMARY KEY(Codigo),
	CONSTRAINT fk_poblaciones FOREIGN KEY(CodigoProvincia) REFERENCES Provincias(Codigo)
);

INSERT INTO Poblaciones (Codigo,Nombre,CodigoProvincia) VALUES ('0','Dos Hermanas','0');
INSERT INTO Poblaciones (Codigo,Nombre,CodigoProvincia) VALUES ('1','Utrera','0');
INSERT INTO Poblaciones (Codigo,Nombre,CodigoProvincia) VALUES ('2','Villablanca','1');
INSERT INTO Poblaciones (Codigo,Nombre,CodigoProvincia) VALUES ('3','Monachil','2');
INSERT INTO Poblaciones (Codigo,Nombre,CodigoProvincia) VALUES ('4','Chipiona','3');
INSERT INTO Poblaciones (Codigo,Nombre,CodigoProvincia) VALUES ('5','Torremolinos','4');
INSERT INTO Poblaciones (Codigo,Nombre,CodigoProvincia) VALUES ('6','Estepona','4');
INSERT INTO Poblaciones (Codigo,Nombre,CodigoProvincia) VALUES ('7','Linares','5');

CREATE TABLE Conductores
(
	Codigo VARCHAR(10),
	Nombre VARCHAR(20),
	Apellidos VARCHAR(20),
	DNI VARCHAR(9),
	Direccion VARCHAR(30),
	Telefono VARCHAR(9),
	CodigoPoblacion VARCHAR(10),
	CONSTRAINT pk_conductores PRIMARY KEY(Codigo),
	CONSTRAINT fk_conductores FOREIGN KEY(CodigoPoblacion) REFERENCES Poblaciones(Codigo),
	CONSTRAINT codiok CHECK(Codigo REGEXP '[0-9]{6,}'),
	CONSTRAINT apellidosok CHECK(UPPER(SUBSTRING(Apellidos,1,1)) = BINARY(SUBSTRING(Apellidos,1,1))),
	CONSTRAINT apellidosok2 CHECK(SUBSTRING(SUBSTRING_INDEX(Apellidos,' ',-1),1,1) = BINARY(UPPER(SUBSTRING(SUBSTRING_INDEX(Apellidos,' ',-1),1,1))))
);

INSERT INTO Conductores (Codigo,Nombre,Apellidos,DNI,Direccion,Telefono,CodigoPoblacion) 
VALUES ('000000','Antonio','Garcia Rodriguez','58647072Q','C/ Tajo, 145','685091321','0');
INSERT INTO Conductores (Codigo,Nombre,Apellidos,DNI,Direccion,Telefono,CodigoPoblacion) 
VALUES ('000001','Jose','Gonzalez Fernandez','83002995C','C/ Judea, 19','657240633','1');
INSERT INTO Conductores (Codigo,Nombre,Apellidos,DNI,Direccion,Telefono,CodigoPoblacion) 
VALUES ('000002','Manuel','Lopez Martinez','98759385F','C/ Espartero, 4','624738904','0');
INSERT INTO Conductores (Codigo,Nombre,Apellidos,DNI,Direccion,Telefono,CodigoPoblacion) 
VALUES ('000003','Francisco','Perez Gomez','62541048T','C/ Marchena, 1D','654782999','1');
INSERT INTO Conductores (Codigo,Nombre,Apellidos,DNI,Direccion,Telefono,CodigoPoblacion) 
VALUES ('000004','David','Martin Jimenez','38186676Y','C/ Ponce de Leon, 3D','783098678','1');
INSERT INTO Conductores (Codigo,Nombre,Apellidos,DNI,Direccion,Telefono,CodigoPoblacion) 
VALUES ('000005','Juan','Ruiz Hernandez','94200021W','C/ Miguel Angel Asturias, 16','657483900','0');
~~~

## Ficheros de configuración

El fichero que tenemos que modificar para que acceda remotamente será:

~~~
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
~~~

Y tenemos que cambiar la `bind-address`  lo siguiente:

~~~
bind-address            = 0.0.0.0
~~~

Reiniciamos el servicio:

~~~
sudo systemctl restart mariadb.service
~~~

## Comprobación a terminal de base de datos

Ya podríamos acceder remotamente con el usuario externo desde nuestra máquina principal con la siguiente ejecución:

~~~
mysql -h 192.168.1.147 -u externo -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 36
Server version: 10.3.27-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
~~~

Y podriamos comprobar que las tablas y la base de datos está perfectamente creada:

~~~
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| prueba             |
+--------------------+
4 rows in set (0.002 sec)

MariaDB [(none)]> use prueba
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [prueba]> show tables;
+------------------+
| Tables_in_prueba |
+------------------+
| Conductores      |
| Poblaciones      |
| Provincias       |
+------------------+
3 rows in set (0.001 sec)
~~~

Pero nuestro objetivo es mostrar dicha información a través de una aplicación web, así que instalaremos `phpMyAdmin`.

## Instalación de phpMyAdmin y requisitos

Tenemos que tener un servidor LAMP, para ello instalaremos los requisitos que nos faltan y activamos el módulo para apache2:

~~~
sudo apt install -y apache2
~~~

También instalaremos php y sus requisitos:

~~~
sudo apt install php php-mysql php7.3 php-bz2 php-mbstring php-zip
sudo systemctl reload apache2
~~~

Vamos a usar el sitio virtual por defecto que trae apache2, así que para ello:

~~~
sudo nano /etc/apache2/sites-available/000-default.conf
~~~

Y lo vamos a dejar de la siguiente forma:

~~~
<VirtualHost *:80>
        ServerName www.app-mysql.org
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
~~~

Ahora copiamos el enlace de descarga de la página principal de php:

![1](/assets/img/posts/servidores-bd/1.png)

Y lo descargamos y descomprimimos en el document root de nuestra máquina:

~~~
cd /var/www/html/

sudo wget https://files.phpmyadmin.net/phpMyAdmin/5.0.4/phpMyAdmin-5.0.4-all-languages.zip

sudo unzip phpMyAdmin-5.0.4-all-languages.zip

sudo rm -rf index.html phpMyAdmin-5.0.4-all-languages.zip

sudo mv phpMyAdmin-5.0.4-all-languages/ phpmyadmin
~~~

## Acceso remoto desde cliente y comprobación de funcionamiento

En el cliente tenemos que cambiar nuestro fichero:

~~~
sudo nano /etc/hosts
~~~

Y dejarlo de la siguiente forma:

~~~
192.168.1.147    www.app-mysql.org
~~~

Y ahora ya podremos acceder a la administración de la base de datos:

![2](/assets/img/posts/servidores-bd/2.png)

Y como vemos tenemos la base de datos de prueba:

![3](/assets/img/posts/servidores-bd/3.png)

Con todo su contenido dentro, el cual podemos modificar:

![4](/assets/img/posts/servidores-bd/4.png)