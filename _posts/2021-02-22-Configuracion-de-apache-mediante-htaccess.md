---
layout: post
title: Configuración de Apache mediante archivo .htaccess
tags: [Apache2, Apache, htaccess]
---
# Introducción

Buenas, en este post explicaremos para que sirve el fichero <code>.htaccess</code> y haremos un pequeño ejercicio.

## ¿Qué es?

Un fichero .htaccess (hypertext access), también conocido como archivo de configuración distribuida, es un fichero especial, popularizado por el Servidor HTTP Apache que nos permite definir diferentes directivas de configuración para cada directorio (con sus respectivos subdirectorios) sin necesidad de editar el archivo de configuración principal de Apache.

Para permitir el uso de los ficheros <code>.htaccess</code> o restringir las directivas que se pueden aplicar usamos la directiva <code>AllowOverride</code>, que puede ir acompañada de una o varias opciones: <code>All, AuthConfig, FileInfo, Indexes, Limit, …</code>

## Alta en hosting gratuito

Nos vamos a dar de alta en un proveedor de hosting gratuito y vamos a configurar el fichero <code>.htaccess</code>.

Hemos escogido un dominio gratuito de <code>000webhost</code>, obviamente no vamos a explicar como darnos de alta, ya que es un proceso bastante sencillo. Al acceder a la administración de subida de archivos tenemos que entrar en la carpeta <code>public_html</code>:

![1](/assets/img/posts/htaccess/1.png)

Aquí ya viene el listado de ficheros por defecto activo en el <code>.htaccess</code> del directorio principal, el cuál afecta a todos sus subdirectorios, por ello lo primero que tenemos que hacer es deshabilitar la opción <code>Indexes</code> del principal:

![2](/assets/img/posts/htaccess/2.png)

Y crearemos otro directorio llamado <code>nas</code> con otro <code>.htaccess</code>:

![3](/assets/img/posts/htaccess/3.png)

En el cuál le añadiremos lo siguiente:

~~~
Options +Indexes
~~~

Y como podemos comprobar nos mostrará los archivos creados:

![4](/assets/img/posts/htaccess/4.png)

## Redirección permanente

Para hacer una redirección a _google_, tendremos que modificar el <code>.htaccess</code> del directorio principal añadiendo la siguiente línea:

~~~
Redirect "/google" "https://www.google.es"
~~~

Y si comprobamos desde nuestra máquina:

~~~
curl https://alejandro-ejercicio6.000webhostapp.com/google

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>302 Found</title>
</head><body>
<h1>Found</h1>
<p>The document has moved <a href="https://www.google.es">here</a>.</p>
</body></html>
~~~

## Autentificación

Vamos a hacer que cuando entremos en <code>http://host.dominio/prohibido</code> nos pida autentificación:

Creamos el directorio <code>prohibido</code>:

![5](/assets/img/posts/htaccess/5.png)

Le creamos algo de contenido:

![6](/assets/img/posts/htaccess/6.png)

Generamos en nuestra máquina un fichero digest:

~~~
htdigest -c digest.txt dominio ale
Adding password for ale in realm dominio.
New password: 
Re-type new password:
~~~

Copiamos su contenido a un nuevo fichero en la carpeta raíz del servidor (para que no sea accesible fácilmente):

![7](/assets/img/posts/htaccess/7.png)

Y para que obtenga el fichero y haga la autenticación, en otro fichero nuevo <code>.htaccess</code>, dentro del directorio <code>prohibido</code> añadimos lo siguiente:

~~~
AuthType Digest
AuthName "dominio" 
AuthUserFile "/storage/ssd5/935/15297935/digest.txt" 
Require valid-user
~~~

La ubicación la hemos tenido que buscar en la página de <code>000web</code>, en los detalles de nuestro sitio:

![8](/assets/img/posts/htaccess/8.png)

No hará falta añadir la habilitación del módulo digest porque ya está activa.

### Comprobación

![9](/assets/img/posts/htaccess/9.png)

Y si ponemos la contraseña correctamente:

![10](/assets/img/posts/htaccess/10.png)