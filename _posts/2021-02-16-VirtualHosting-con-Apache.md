---
layout: post
title: VirtualHosting con Apache
tags: [all, Apache2, Apache, Servidor, VirtualHosting, Sitios virtuales]
---
# Introducción

Buenas, en esta práctica vamos a hacer una pequeña introducción al Virtualhosting para posteriormente la creación de dos sitios virtuales.

## Introducción al VirtualHosting

El término Hosting Virtual se refiere a hacer funcionar más de un sitio web (tales como <code>www.company1.com</code> y <code>www.company2.com</code>) en una sola máquina. Los sitios web virtuales pueden estar “basados en direcciones IP”, lo que significa que cada sitio web tiene una dirección IP diferente, o “basados en nombres diferentes”, lo que significa que con una sola dirección IP están funcionando sitios web con diferentes nombres (de dominio). Apache fue uno de los primeros servidores web en soportar hosting virtual basado en direcciones IP.

El servidor web Apache2 se instala por defecto con un host virtual. La configuración de este sitio la podemos encontrar en:

~~~
/etc/apache2/sites-available/000-default.conf
~~~

Cuyo contenido podemos ver:

~~~
<VirtualHost *:80>
    #ServerName www.example.com	
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html	
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined	
</VirtualHost>
~~~

Y por defecto este sitio virtual está habilitado, por lo que podemos comprobar que existe un enlace simbólico a este fichero en el directorio <code>/etc/apache2/sites-enables</code>:

~~~
lrwxrwxrwx 1 root root   35 Oct  3 15:24 000-default.conf -> ../sites-available/000-default.conf
~~~

Podemos habilitar o deshabilitar nuestros host virtuales utilizando los siguientes comandos:

~~~
a2ensite
a2dissite
~~~

En el fichero de configuración general <code>/etc/apache2/apache2.conf</code> nos encontramos las opciones de configuración del directorio padre del indicado en la directiva DocumentRoot (suponemos que todos los host virtuales van a estar guardados en subdirectorios de este directorio):

~~~
...
<Directory /var/www/>
	Options Indexes FollowSymLinks
	AllowOverride None
	Require all granted
</Directory>
...
~~~

## Configuración de VirtualHosting

El objetivo de esta práctica es la puesta en marcha de dos sitios web utilizando el mismo servidor web apache. Hay que tener en cuenta lo siguiente:

- Cada sitio web tendrá nombres distintos.
- Cada sitio web compartirán la misma dirección IP y el mismo puerto (80).

Queremos construir en nuestro servidor web apache dos sitios web con las siguientes características:

- El nombre de dominio del primero será <code>www.pagina1.com</code>, su directorio base será <code>/var/www/pagina1</code> y contendrá una página llamada index.html, donde sólo se verá una bienvenida a la página.

- En el segundo sitio vamos a crear una página llamada <code>www.pagina2.com</code>, y su directorio base será <code>/var/www/pagina2</code>. En este sitio sólo tendremos una página inicial index.html.

Para conseguir estos dos sitios virtuales debes seguir los siguientes pasos:

## Instalación de apache2

Instalamos Apache2 si no lo tenemos instalado:

~~~
sudo apt install apache2
~~~

## Desactivamos el sitio por defecto

~~~
cd /etc/apache2/sites-available/
a2dissite 000-default.conf
~~~

## Copiamos la página por defecto

~~~
cp 000-default.conf pagina1.conf
cp 000-default.conf pagina2.conf
~~~

## Cambiamos configuración

Entramos en la configuración de la página:

~~~
sudo nano pagina1.conf
~~~

Y lo dejamos de la siguiente forma:

~~~
<VirtualHost *:80>
	ServerName www.pagina1.com
	DocumentRoot /var/www/pagina1
	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
~~~

Igualmente en la página 2:

~~~
nano pagina2.conf
~~~

Y la modificamos:

~~~
<VirtualHost *:80>
    ServerName www.pagina2.com
    DocumentRoot /var/www/pagina2
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
~~~

## Creamos index

Creamos el <code>index.html</code> de las dos páginas:

~~~
cd /var/www/
~~~

~~~
mkdir pagina1
mkdir pagina2
~~~

~~~
cp html/index.html pagina1
cp html/index.html pagina2
~~~


## Activamos los sitios

~~~
cd /etc/apache2/sites-available/
sudo a2ensite pagina1.conf
sudo a2ensite pagina2.conf
~~~

## Reiniciamos apache2

~~~
sudo systemctl restart apache2
~~~

## Prueba de funcionamiento

Para que funcione en el cliente debemos de añadir las siguientes líneas al fichero <code>/etc/hosts</code>:

~~~
172.22.8.26	www.pagina1.com
172.22.8.26	www.pagina2.com
~~~

Y comprobamos:

![img1](/assets/img/posts/virtualhosting/pagina1.png)

![img2](/assets/img/posts/virtualhosting/pagina2.png)