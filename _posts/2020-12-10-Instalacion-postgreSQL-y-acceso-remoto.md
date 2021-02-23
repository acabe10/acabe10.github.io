---
layout: post
title: Instalación postgreSQL y acceso remoto
tags: [all, PostgreSQL, BBDD]
---
# Introducción

Buenas, en este post vamos a instalar y configurar PostgreSQL para su acceso remoto.

Partiendo de un escenario en el cuál hemos creado una máquina `Debian Buster` con `Vagrant`, vamos a instalar y acceder remotamente desde nuestra máquina anfitriona a un servicio de `Postgres`.

## Instalación Postgres

Instalamos `postgresql`:

~~~
$ sudo apt install postgresql
~~~

Añadimos una contraseña al nuevo usuario que se ha creado:

~~~
$ sudo passwd postgres
~~~

## Creación de usuario y base de datos poblada

Accedemos al usuario postgres:

~~~
$ su postgres
~~~

Creamos un usuario:

~~~
$ createuser -S -R -D -l externo
~~~

Accedemos a postgres para dar una contraseña a dicho usuario:

~~~
$ psql
psql (11.9 (Debian 11.9-0+deb10u1))
Type "help" for help.

postgres=# ALTER USER externo WITH ENCRYPTED PASSWORD 'externo';
~~~

Editamos el fichero:

~~~
sudo nano /etc/postgresql/11/main/pg_hba.conf
~~~

Para poder acceder mediante la contraseña desde nuestro equipo y cambiamos la siguiente línea:

~~~
local   all             all                                     peer
~~~

Por esta otra:

~~~
local   all             all                                     md5
~~~

Reiniciamos el servicio:

~~~
sudo systemctl restart postgresql
~~~

Creamos una base de datos:

~~~
su postgres
Password: 
postgres@server:/home/vagrant$ createdb prueba
~~~

La poblamos de datos:

~~~
postgres@server:/home/vagrant$ psql -U externo -d prueba
psql (11.9 (Debian 11.9-0+deb10u1))
Type "help" for help.

prueba=# CREATE TABLE Provincias
(
	Codigo 		VARCHAR(10),
	Nombre 		VARCHAR(20),
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
	Codigo 			VARCHAR(10),
	Nombre 			VARCHAR(20),
	CodigoProvincia	VARCHAR(10),
	CONSTRAINT pk_poblaciones PRIMARY KEY(Codigo),
	CONSTRAINT fk_poblaciones FOREIGN KEY(CodigoProvincia) REFERENCES Provincias
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
	Codigo 			VARCHAR(10),
	Nombre 			VARCHAR(20),
	Apellidos 		VARCHAR(20),
	DNI				VARCHAR(9),
	Direccion 		VARCHAR(30),
	Telefono 		VARCHAR(9),
	CodigoPoblacion	VARCHAR(10),
	CONSTRAINT pk_conductores PRIMARY KEY(Codigo),
	CONSTRAINT fk_conductores FOREIGN KEY(CodigoPoblacion) REFERENCES Poblaciones,
	CONSTRAINT codiok CHECK(Codigo ~ '[0-9]{6,}'),
	CONSTRAINT apellidosok CHECK(INITCAP(Apellidos) = Apellidos)
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

Para que se pueda acceder desde la red local tendremos que modificar dos archivos de configuración, el primero:

~~~
sudo nano /etc/postgresql/11/main/postgresql.conf
~~~

Y descomentamos y ponemos de la siguiente forma lo siguiente:

~~~
listen_addresses = '*'
~~~

El otro archivo de configuración:

~~~
sudo nano /etc/postgresql/11/main/pg_hba.conf
~~~

Y modificamos para que quede de la siguiente forma la línea de `IPv4 local connections`

~~~
# IPv4 local connections:
host    all             all             192.168.1.0/24            md5
~~~

Reiniciamos el servicio:

~~~
sudo systemctl restart postgresql
~~~

## Acceso remoto desde cliente y comprobación de funcionamiento

Para poder acceder desde nuestra máquina anfitriona tendremos que tener instalado un paquete de cliente:

~~~
sudo apt install postgresql-client
~~~

Y simplemente con realizar el siguiente comando tendremos el acceso:

~~~
$ psql -h 192.168.1.177 -U externo -d prueba
Contraseña para usuario externo: 
psql (11.9 (Debian 11.9-0+deb10u1))
conexión SSL (protocolo: TLSv1.3, cifrado: TLS_AES_256_GCM_SHA384, bits: 256, compresión: desactivado)
Digite «help» para obtener ayuda.

prueba=> \dt
          Listado de relaciones
 Esquema |   Nombre    | Tipo  |  Dueño   
---------+-------------+-------+----------
 public  | conductores | tabla | postgres
 public  | poblaciones | tabla | postgres
 public  | provincias  | tabla | postgres
(3 filas)

prueba=> SELECT codigo,nombre,dni FROM CONDUCTORES;
 codigo |  nombre   |    dni    
--------+-----------+-----------
 000000 | Antonio   | 58647072Q
 000001 | Jose      | 83002995C
 000002 | Manuel    | 98759385F
 000003 | Francisco | 62541048T
 000004 | David     | 38186676Y
 000005 | Juan      | 94200021W
(6 filas)
~~~