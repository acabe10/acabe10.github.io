---
layout: post
title: Instalación SQL Developer sobre Windows
tags: [all, SQL Developer, Windows, BBDD]
---

# Instalación de SQL Developer sobre Windows

Buenas, en este post vamos a partir de un escenario que tenemos montado en nuestra red local en el cual tenemos una máquina Debian con un Oracle 12c instalado, y también tenemos una máquina Windows en la cuál vamos a instalar SQL Developer.

## Descarga e instalación

Nos tendremos que ir a la página oficial de Oracle y descargarnos el instalador de SQL Developer:

![15](/assets/img/posts/servidores-bd/5.png)

Cuando se descargue lo descomprimimos y dentro de la carpeta, pulsando en `sqldeveloper` comenzará la instalación:

![6](/assets/img/posts/servidores-bd/6.png)

## Funcionamiento

Una vez instalado, se nos abrirá la siguiente ventana, que es la principal del programa:

![7](/assets/img/posts/servidores-bd/7.png)

Para poder crear una conexión nueva, pulsamos en añadir nueva conexión o en la pestaña de la izquierda en el simbolo "+", y añadimos los datos que nos harán falta y probamos la conexión:

![8](/assets/img/posts/servidores-bd/8.png)

`Name`: nombre para identificar la conexión.
`Usuario`: usuario que administra la base de datos a la que vamos a acceder.
`Contraseña`: contraseña del usuario anterior.
`Nombre del Host`: en nuestro caso, al ser una conexión remota, la ip del equipo al que queremos acceder.
`Puerto`: Oracle por defecto usa el 1521, a no ser que el administrador lo haya cambiado.
`SID`: nombre de la base de datos que creamos cuando instalamos oracle por primera vez, en nuestro caso la llamamos base1.

Una vez que nos diga que está correta, pulsamos en `Conectar` y al cabo de unos segundos nos aparecerá en la pestaña de la izquierda:

![9](/assets/img/posts/servidores-bd/9.png)

Ahora ya podremos navegar por las tablas que tiene permiso el usuario el cuál hemos introducido anteriormente:

![10](/assets/img/posts/servidores-bd/10.png)

Y realizar bastantes gestiones con dicho programa:

![11](/assets/img/posts/servidores-bd/11.png)