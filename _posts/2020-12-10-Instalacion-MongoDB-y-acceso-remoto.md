---
layout: post
title: Instalación MongoDB y acceso remoto
tags: [all, MongoDB, BBDD]
---
# Introducción

Buenas, en este post, partiendo de un escenario igual que el anterior instalaremos MongoDB y permitiremos su acceso remoto desde la red local.

## Instalación

`MongoDB` no lo tenemos en los repositorios oficiales de `Debian`, así que añadiremos el repositorio y su clave de `MongoDB` a nuestro equipo:

~~~
# wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -

# echo "deb http://repo.mongodb.org/apt/debian buster/mongodb-org/4.4 main" >> /etc/apt/sources.list
~~~

Ahora instalamos `MongoDB`:

~~~
$ sudo apt update && sudo apt install mongodb-org
~~~

Iniciamos el servicio:

~~~
sudo systemctl start mongod
~~~

Comprobamos:

~~~
$ mongo
MongoDB shell version v4.4.2
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("13b3cd26-50d2-4499-9905-a33493091e44") }
MongoDB server version: 4.4.2
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	https://docs.mongodb.com/
Questions? Try the MongoDB Developer Community Forums
	https://community.mongodb.com
---
The server generated these startup warnings when booting: 
        2020-12-09T12:09:58.134+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
        2020-12-09T12:09:58.694+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
---
---
        Enable MongoDB's free cloud-based monitoring service, which will then receive and display
        metrics about your deployment (disk utilization, CPU, operation statistics, etc).

        The monitoring data will be available on a MongoDB website with a unique URL accessible to you
        and anyone you share the URL with. MongoDB may use this information to make product
        improvements and to suggest MongoDB products and deployment options to you.

        To enable free monitoring, run the following command: db.enableFreeMonitoring()
        To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---
>
~~~

Al comienzo de instalar mongo nos lo crea sin usuario y sin contraseña, para hacerlo seguro en el siguiente apartado crearemos un usuario y una contraseña para poder acceder.

## Creación de usuario y base de datos poblada

Primero vamos a crear un usuario administrador para todas las bases de datos, para ello entramos en `mongo` como anteriormente y:

~~~
> use admin

> db.createUser(
...   {
...     user: "ale",
...     pwd: "ale",
...     roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]
... }
... )
Successfully added user: {
	"user" : "ale",
	"roles" : [
		{
			"role" : "userAdminAnyDatabase",
			"db" : "admin"
		},
		"readWriteAnyDatabase"
	]
}
~~~

Con el usuario ya creado, necesitaremos que mongo nos pida una contraseña al comienzo de iniciarlo, para ello editamos su fichero de configuración:

~~~
sudo nano /etc/mongod.conf
~~~

Y añadimos lo siguiente:

~~~
security:
  authorization: enabled
~~~

Reiniciamos el servicio:

~~~
sudo systemctl restart mongod
~~~

Ahora podemos acceder perfectamente:

~~~
$ mongo --authenticationDatabase "admin" -u "ale" -p "ale"
MongoDB shell version v4.4.2
connecting to: mongodb://127.0.0.1:27017/?authSource=admin&compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("26cab46d-80ea-4541-ad2b-93822ee016f9") }
MongoDB server version: 4.4.2
>
~~~

Vamos a crear la base de datos:

~~~
> use prueba
switched to db prueba
~~~

Y añadimos el usuario que la administrará:

~~~
> db.createUser(
...    {
...      user: "user_prueba",
...      pwd: "password_prueba",
...      roles: ["dbOwner"]
...    }
...  )
Successfully added user: { "user" : "user_prueba", "roles" : [ "dbOwner" ] }
~~~

Nos metemos con el usuario nuevo:

~~~
mongo --authenticationDatabase "prueba" -u "user_prueba" -p "password_prueba"
~~~

Y añadimos algunos datos para probar posteriormente:

~~~
> db
test
> use prueba
switched to db prueba
> db.conductores.insertOne(
...     {
...       dni: "490302124F",
...       nombre: "Alejandro",
...       direccion: "C/ Miguel Ángel",
...       telefono: "618321555",
...       fechanacimiento: new Date("1994-01-28")
...     }
...   )
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5fd0e1e43149572bad0d11d6")
}
> db.conductores.insertOne(
...     {
...       dni: "18232747A",
...       nombre: "Carmen María Martín",
...       direccion: "Hola , 17",
...       telefono: "691777555",
...       fechanacimiento: new Date("1994-05-07")
...     }
...   )
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5fd0e1f03149572bad0d11d7")
}
> db.camiones.insertOne(
...      {
...        matricula: "3656HJK",
...        fechamatriculacion: new Date("1994-02-07")
...      }
...    )
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5fd0e1f73149572bad0d11d8")
}
> db.camiones.insertOne(
...      {
...        matricula: "4333FKB",
...        fechamatriculacion: new Date("1994-01-07")
...      }
...    )
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5fd0e1fc3149572bad0d11d9")
}
> db.provincias.insertOne(
...       {
...         nombre: "Sevilla",
...         habitantes: "150"
...       }
...     )
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5fd0e2003149572bad0d11da")
}
> db.provincias.insertOne(
...       {
...         nombre: "Granada",
...         habitantes: "290"
...       }
...     )
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5fd0e2043149572bad0d11db")
}
~~~

## Ficheros de configuración

Para que podamos acceder remotamente, tenemos que editar su fichero de configuración:

~~~
sudo nano /etc/mongod.conf
~~~

Y editamos la siguiente líneas:

~~~
bindIp: 0.0.0.0
~~~

Reiniciamos el servicio:

~~~
sudo systemctl restart mongod
~~~

También vamos a modificar el usuario `user prueba` para que pueda acceder desde el exterior, para ello entramos en mongo como admin:

~~~
mongo --authenticationDatabase "admin" -u "ale" -p "ale"
~~~

Y le damos tal permiso al usuario:

~~~
> use prueba
switched to db prueba
> db.updateUser(
     "user_prueba",
   {
     authenticationRestrictions: [ { clientSource: ["192.168.1.0/24"] } ]
   }
 )
~~~

## Acceso remoto desde cliente y comprobación de funcionamiento

Hemos habilitado para que funcione sólo desde la red externa, así que lo probaremos:

~~~
mongo -u user_prueba -p password_prueba 192.168.1.150/prueba
MongoDB shell version v4.4.2
connecting to: mongodb://192.168.1.150:27017/prueba?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("12eda456-52cb-42a5-bd32-5d4cf2357d9e") }
MongoDB server version: 4.4.2
> 
~~~

También podemos ver que se muestran las colecciones:

~~~
> show collections
camiones
conductores
provincias
~~~

Y que nos muestra su contenido:

~~~
> db.conductores.find().pretty()
{
	"_id" : ObjectId("5fd0e1e43149572bad0d11d6"),
	"dni" : "490302124F",
	"nombre" : "Alejandro",
	"direccion" : "C/ Miguel Ángel",
	"telefono" : "618321555",
	"fechanacimiento" : ISODate("1994-01-28T00:00:00Z")
}
{
	"_id" : ObjectId("5fd0e1f03149572bad0d11d7"),
	"dni" : "18232747A",
	"nombre" : "Carmen María Martín",
	"direccion" : "Hola , 17",
	"telefono" : "691777555",
	"fechanacimiento" : ISODate("1994-05-07T00:00:00Z")
}
~~~
~~~
> db.provincias.find().pretty()
{
	"_id" : ObjectId("5fd0e2003149572bad0d11da"),
	"nombre" : "Sevilla",
	"habitantes" : "150"
}
{
	"_id" : ObjectId("5fd0e2043149572bad0d11db"),
	"nombre" : "Granada",
	"habitantes" : "290"
}
~~~