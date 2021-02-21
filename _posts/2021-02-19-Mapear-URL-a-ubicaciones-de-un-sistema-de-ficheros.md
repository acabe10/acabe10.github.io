---
layout: post
title: Mapear URL a ubicaciones de un sistema de ficheros
tags: [Mapeo, Apache2, Apache, Servidor, Sitios virtuales]
---
# Introducción

Buenas, en este post vamos a explicar como redireccionar direcciones dentro de apache2 y como mostrar ficheros alojados en otros directorios con varios ejemplos.

Vamos a suponer que ya tenemos instalado un sitio virtual llamado <code>www.mapeo.com</code> y su <code>DocumentRoot</code> es <code>/srv/mapeo</code>.

## Opciones de directorios

Cuando indicamos la configuración de un servidor servidor por apache, por ejemplo con la directiva Directory, podemos indicar algunas opciones con la directiva Options. Algunas de las opciones que podemos indicar son las siguientes:

* All: Se pondrán todas las options menos MultiViews.
* FollowSymLinks: Se seguirán los enlaces simbólicos.
* Indexes: Si no hay Index.html o similares, devolverá una lista formateada del directorio.
* MultiViews: Se permite el uso de contenido negociado, es decir, el servidor elige el mejor contenido a mostrar dependiendo de la configuración del navegador, por ejemplo, el idioma.
* SymLinksIfOwnerMatch: Se seguirán los enlaces simbólicos cuando el propietario del enlace y del archivo o directorio sea el mismo.
* ExecCGI: Se permite la ejecución de scripts CGI usando el mod_cgi.

## Redireccionar www.mapeo.com

Vamos a hacer un redirect para que cada vez que se acceda a <code>www.mapeo.com</code> nos muestre <code>www.mapeo.com/principal</code>. Para ello vamos al archivo de configuración de nuestro VirtualHost, <code>/etc/apache2/sites-available/mapeo.conf</code> y lo dejamos de la siguiente forma:

~~~
<VirtualHost *:80>
        ServerName www.mapeo.com
        DocumentRoot /srv/mapeo/
        Redirect "/index.html" "/principal/"
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
~~~

Comprobamos que funciona correctamente:

![principal](/assets/img/posts/mapeo/principal.png)

## Cambiando opciones de directorio

Para que no se muestren la lista de los ficheros, que no se permitan los enlaces simbólicos y no se permita la negociación de contenido, añadiremos lo siguiente al fichero _/etc/apache2/sites-available_:

~~~
<Directory /srv/mapeo/principal>
        Options -Indexes -FollowSymLinks -MultiViews
</Directory>
~~~

## Mostrar contenido de otro directorio mediante enlace simbólico

Para que nos muestre el contenido de <code>/home/vagrant/doc</code> cada vez que accedamos a <code>www.mapeo.com/principal/documentos</code> crearemos un enlace simbólico, para ello:

~~~
mkdir /home/vagrant/doc
ln -s /home/vagrant/doc /srv/mapeo/principal/documentos
~~~

También tendremos que modificar el fichero <code>/etc/apache2/sites-available/mapeo.conf</code> al cuál le añadiremos las siguientes <code>options</code> en nuestro directorio principal:

~~~
<Directory /srv/mapeo/principal>
        Options +Indexes +SymLinksIfOwnerMatch -FollowSymLinks -MultiViews
</Directory>
~~~

La opción <code>SymLinksIfOwnerMatch</code> permitirá el seguimiento de los enlaces sólo si el creador del enlace es el mismo usuario que el del directorio o fichero destino. Vamos a mostrar que pasa si tenemos el fichero con otro propietario:

~~~
root@serverweb:/srv/mapeo/principal# ls -la
total 20
drwxr-xr-x 2 www-data www-data  4096 Oct 21 18:49 .
drwxr-xr-x 3 www-data www-data  4096 Oct 20 15:39 ..
lrwxrwxrwx 1 www-data www-data    17 Oct 21 18:49 documentos -> /home/vagrant/doc
-rw-r--r-- 1 www-data www-data 10692 Oct 20 16:42 index.html
~~~

Como vemos el propietario es el usuario <code>www-data</code>, si nos vamos a la página <code>documentos</code> de nuestro servidor no nos mostrará nada porque el propietario de <code>/home/vagrant/doc</code> es el usuario _vagrant_:

![otrousuario](/assets/img/posts/mapeo/otrousuario.png)

Pero en cambio si cambiamos el propietario del enlace simbólico al usuario _vagrant_:

~~~
root@serverweb:/srv/mapeo/principal# chown -R vagrant:vagrant documentos
root@serverweb:/srv/mapeo/principal# ls -la
total 20
drwxr-xr-x 2 www-data www-data  4096 Oct 21 18:49 .
drwxr-xr-x 3 www-data www-data  4096 Oct 20 15:39 ..
lrwxrwxrwx 1 vagrant  vagrant     17 Oct 21 18:49 documentos -> /home/vagrant/doc
-rw-r--r-- 1 www-data www-data 10692 Oct 20 16:42 index.html
~~~

Nos dará la información que queremos:

![usuariocorrecto](/assets/img/posts/mapeo/usuariocorrecto.png)

## Páginas de error personalizadas

Para poder hacer páginas de error personalizadas vamos a crear un directorio en el cuál las vamos a guardar:

~~~
mkdir /srv/error
~~~

Y crearemos dos html para los errores 403 y 404. Tendremos que dar permisos a ese directorio en el fichero de configuración de nuestro VirtualHost:

~~~
nano /etc/apache2/sites-available/mapeo.conf
~~~

Añadimos las siguientes líneas:

~~~
<Directory /srv/error>
                Options FollowSymLinks
                AllowOverride None
                Require all granted
</Directory>
~~~

Y con la directiva <code>ErrorDocument</code> activaremos esos dos html en el mismo fichero anterior, añadiendo lo siguiente:

~~~
ErrorDocument 404 /srv/error/err404.html
ErrorDocument 403 /srv/error/err403.html
~~~

Comprobamos con página no encontrada:

![noencontrada](/assets/img/posts/mapeo/noencontrada.png)

Y con página de acceso denegado:

![denegado](/assets/img/posts/mapeo/denegado.png)