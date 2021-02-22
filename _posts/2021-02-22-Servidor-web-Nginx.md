---
layout: post
title: Servidor web Nginx
tags: [Nginx, Servidor web, Virtualhosting, Mapeo, Autentificación, Autorización, Control acceso]
---
# Introducción

Buenas, en este post vamos a instalar y configurar un servidor web con Nginx siguiendo los siguientes puntos:

* Virtual Hosting.
* Mapeo de URL.
* Autentificación, Autorización, y Control de Acceso.

Vamos a realizarlo en una máquina que tenemos en un cloud privado, podríamos realizarlo también en una máquina con vagrant.

## Instalación

Instalamos <code>nginx</code> en la máquina del cloud:

~~~
apt install nginx
~~~

Modificamos algo la página que viene por defecto:

~~~
nano /var/www/html/index.nginx-debian.html
~~~

Y accedemos por el navegador no sin antes añadir una regla que abra el puerto 80 en nuestro proyecto del cloud:

![1](/assets/img/posts/nginx/1.png)

## Virtual hosting

Queremos que nuestro servidor web ofrezca dos sitios web, teniendo en cuenta lo siguiente:

1. Cada sitio web tendrá nombres distintos.
2. Cada sitio web compartirán la misma dirección IP y el mismo puerto (80).

Los dos sitios web tendrán las siguientes características:

* El nombre de dominio del primero será <code>www.iesgn.org</code>, su directorio base será <code>/srv/www/iesgn</code> y contendrá una página llamada <code>index.html</code>, donde sólo se verá una bienvenida a la página del Instituto Gonzalo Nazareno.
* En el segundo sitio vamos a crear una página donde se pondrán noticias por parte de los departamentos, el nombre de este sitio será <code>departamentos.iesgn.org</code>, y su directorio base será <code>/srv/www/departamentos</code>. En este sitio sólo tendremos una página inicial <code>index.html</code>, dando la bienvenida a la página de los departamentos del instituto.

Vamos a crear las dos carpetas que alojarán los sitios en nginx:

~~~
sudo mkdir -p /srv/www/www.iesgn.org
sudo mkdir /srv/www/departamentos.iesgn.org
~~~

Le creamos un <code>html</code> con algo de contenido en cada directorio:

~~~
# echo "<p>Bienvenido al instituto Gonzalo Nazareno</p" > /srv/www/www.iesgn.org/index.html

# echo "<p>Bienvenido a los DEPARTAMENTOS del IES Gonzalo Nazareno</p" > /srv/www/departamentos.iesgn.org/index.html
~~~

Borramos el sitio por defecto que trae <code>nginx</code>:

~~~
sudo rm /etc/nginx/sites-enabled/default
~~~

Creamos dos nuevos ficheros de configuración para cada sitio y le damos algo de contenido:

~~~
sudo nano /etc/nginx/sites-available/www.iesgn.org

server {
        listen 80;
        server_name www.iesgn.org iesgn.org;
        root /srv/www/www.iesgn.org;
        index index.html;
        location / {
                try_files $uri $uri/ =404;
        }
}
~~~

~~~
sudo nano /etc/nginx/sites-available/departamentos.iesgn.org

server {
        listen 80;
        server_name departamentos.iesgn.org iesgn.org;
        root /srv/www/departamentos.iesgn.org;
        index index.html;
        location / {
                try_files $uri $uri/ =404;
        }
}
~~~

Y creamos dos enlaces simbólicos para que estén activos:

~~~
sudo ln -s /etc/nginx/sites-available/www.iesgn.org /etc/nginx/sites-enabled/www.iesgn.org

sudo ln -s /etc/nginx/sites-available/departamentos.iesgn.org /etc/nginx/sites-enabled/departamentos.iesgn.org
~~~

Damos los permisos al usuario <code>www-data</code>:

~~~
sudo chown -R www-data. /srv/www/
~~~

Reiniciamos el servicio de nginx:

~~~
sudo systemctl restart nginx
~~~

Y solo nos faltaría añadir en el fichero <code>/etc/hosts</code> del cliente las siguientes líneas:

~~~
172.22.200.123 www.iesgn.org
172.22.200.123 departamentos.iesgn.org
~~~

Comprobamos:

![12](/assets/img/posts/nginx/2.png)

![3](/assets/img/posts/nginx/3.png)

## Redirección

Vamos a añadirle las siguientes características a nuestro sitio:

* Cuando se entre a la dirección <code>www.iesgn.org</code> se redireccionará automáticamente a <code>www.iesgn.org/principal</code>, donde se mostrará el mensaje de bienvenida. En el directorio principal no se permite ver la lista de los ficheros, no se permite que se siga los enlaces simbólicos y no se permite negociación de contenido.

Creamos el directorio <code>principal</code> y movemos el directorio de bienvenida:

~~~
sudo mkdir /srv/www/www.iesgn.org/principal
sudo mv /srv/www/www.iesgn.org/index.html /srv/www/www.iesgn.org/principal/
~~~

Damos los permisos al usuario <code>www-data</code>:

~~~
sudo chown -R www-data. /srv/www/
~~~

Si no tuvieramos ningún <code>index.html</code> en la carpeta <code>www.iesgn.org</code> automáticamente buscaría dentro de la carpeta <code>principal</code> y nos mostraría su <code>index.html</code>.

Y modificamos el archivo de configuración de nuestro VirtualHost añadiendole el <code>rewrite</code>:

~~~
server {
        listen 80;
        server_name www.iesgn.org iesgn.org;

        root /srv/www/www.iesgn.org;
        index index.html;

        rewrite ^/$ principal/ permanent;

        location / {
                try_files $uri $uri/ =404;
        }
}
~~~

Con <code>return</code> es complicado hacer que funcione correctamente, así que opte por <code>rewrite</code>.

Para que no se permita la lista de los ficheros, que se siga los enlaces simbólicos y no se permita negociación de contenido añadimos otro <code>location</code>:

~~~
location /principal {
          autoindex off;
          disable_symlinks on;
}
~~~

No he encontrado niguna opción que sustituya completamente a las <code>Multiviews</code> de <code>Apache2</code>, lo más parecido que hay es el <code>try files</code> que ya viene por defecto si le añadimos lo siguiente:

~~~
location ~ ^(/.+)/ {
    try_files $uri $1?$args $1.php?$args;
}
~~~

## Alias

Si accedes a la página <code>www.iesgn.org/principal/documentos</code> se visualizarán los documentos que hay en <code>/srv/doc</code>. Por lo tanto se permitirá el listado de ficheros y el seguimiento de enlaces simbólicos siempre que sean a ficheros o directorios cuyo dueño sea el usuario.

Creamos la carpeta <code>doc</code> en <code>srv</code>:

~~~
mkdir /srv/doc
~~~

Y creamos dos ficheros de prueba:

~~~
echo "dentro de prueba1" > /srv/doc/prueba1.txt
echo "dentro de prueba2" > /srv/doc/prueba2.txt
~~~

Creamos otro <code>location</code> en el fichero de configuración de nuestro virtualhost:

~~~
nano /etc/nginx/sites-available/www.iesgn.org

location /principal/documentos {
                alias /srv/doc;
                autoindex on;
                disable_symlinks if_not_owner;
}
~~~

Y comprobamos que se muestran los ficheros:

![4](/assets/img/posts/nginx/4.png)

Para comprobar que funciona el seguimiento de enlaces simbólicos si el dueño es el mismo creamos otro fichero de prueba:

~~~
echo "dentro de prueba3" > /home/debian/prueba3.txt
~~~

Y creamos el enlace simbólico a ese fichero:

~~~
ln -s /home/debian/prueba3.txt /srv/doc/prueba3.txt
~~~

Si ponemos un propietario diferente del enlace que del fichero tal que así:

~~~
chown -R www-data. /home/debian/prueba3.txt
~~~
~~~
ls -l /srv/doc/
total 8
-rw-r--r-- 1 www-data www-data 18 Nov  8 10:24 prueba1.txt
-rw-r--r-- 1 www-data www-data 18 Nov  8 10:24 prueba2.txt
lrwxrwxrwx 1 www-data www-data 24 Nov  8 11:14 prueba3.txt -> /home/debian/prueba3.txt
~~~

~~~
chown -R debian. /home/debian/prueba3.txt

ls -l /home/debian/prueba3.txt 
-rw-r--r-- 1 debian debian 18 Nov  8 11:14 /home/debian/prueba3.txt
~~~

E intentamos acceder al fichero no podremos:

![5](/assets/img/posts/nginx/5.png)

## Mensajes de error personalizados

Para poder hacer páginas de error personalizadas vamos a crear un directorio en el cuál las vamos a guardar:

~~~
sudo mkdir /srv/www/www.iesgn.org/error
~~~

Creamos las páginas personalizadas:

~~~
echo "<h1>Pagina prohibida</h1>" > /srv/www/www.iesgn.org/error/err403.html
echo "<h1>Pagina no encontrada</h1>" > /srv/www/www.iesgn.org/error/err404.html
~~~

Ahora añadiremos la ruta a seguir en el fichero de configuración:

~~~
sudo nano /etc/nginx/sites-available/www.iesgn.org
~~~

Añadimos:

~~~
error_page 403 /error/err403.html;
error_page 404 /error/err404.html;
~~~

Y si comprobamos página no encontrada:

![6](/assets/img/posts/nginx/6.png)

Y página no permitida:

![7](/assets/img/posts/nginx/7.png)

## Control de acceso

Vamos a añadir otra máquina a nuestro escenario conectada a la red interna. A la URL <code>departamentos.iesgn.org/intranet</code> sólo se debe tener acceso desde el cliente de la red local, y no se pueda acceder desde la anfitriona por la red pública. A la URL <code>departamentos.iesgn.org/internet</code>, sin embargo, sólo se debe tener acceso desde la anfitriona por la red pública, y no desde la red local.

Partiendo del siguiente escenario en <code>OpenStack</code>:

![8](/assets/img/posts/nginx/8.png)

Creamos los directorios <code>intranet</code> y <code>internet</code>:

~~~
mkdir /srv/www/departamentos.iesgn.org/intranet
mkdir /srv/www/departamentos.iesgn.org/internet
~~~

Y le ponemos un index:

~~~
echo "Esto es INTRANET" > /srv/www/departamentos.iesgn.org/intranet/index.html
echo "Esto es INTERNET" > /srv/www/departamentos.iesgn.org/internet/index.html
~~~

Ahora editamos el fichero de configuración de nuestro VirtualHost:

~~~
nano /etc/nginx/sites-available/departamentos.iesgn.org
~~~

Y añadimos lo siguiente:

~~~
location /intranet {
                allow 192.168.200.0/24;
                deny all;
}

location /internet {
                allow 172.23.0.0/16;
                deny all;
}
~~~

Hemos añadido la red que tenemos con la VPN para poder acceder.

Y comprobamos si accedemos desde la red pública a internet:

![9](/assets/img/posts/nginx/9.png)

Desde la red pública a intranet:

![10](/assets/img/posts/nginx/10.png)

Comprobamos desde el cliente acceder a la intranet:

~~~
lynx departamentos.iesgn.org/intranet
~~~

Y comprobamos que funciona correctamente:

![11](/assets/img/posts/nginx/11.png)

Y comprobamos entrar en internet desde el cliente local:

~~~
lynx departamentos.iesgn.org/internet
~~~

Y observamos que no nos lo permite:

![12](/assets/img/posts/nginx/12.png)

## Autentificación básica

Vamos a limitar el acceso a la URL <code>departamentos.iesgn.org/secreto</code> y comprobaremos las cabeceras de los mensajes que se intercambian entre el servidor y el cliente.

Creamos el directorio <code>secreto</code> y le añadimos algo de contenido:

~~~
mkdir /srv/www/departamentos.iesgn.org/secreto
~~~

~~~
echo "Esto es SECRETO" > /srv/www/departamentos.iesgn.org/secreto/index.html
~~~

Instalamos las utilidades de apache para poder crear el archivo de contraseñas:

~~~
apt install apache2-utils
~~~

Creamos el archivo de contraseñas:

~~~
htpasswd -c /etc/nginx/.htpasswd ale
New password: 
Re-type new password: 
Adding password for user ale
~~~

Ahora editamos el archivo de configuración del virtualhost:

~~~
location /secreto {
                auth_basic "Secreto";
                auth_basic_user_file /etc/nginx/.htpasswd;
}
~~~

Y comprobamos que nos pide la contraseña al intentar acceder:

![13](/assets/img/posts/nginx/13.png)

Y cuando ingresamos la contraseña nos muestra el contenido:

![14](/assets/img/posts/nginx/14.png)

## Control de acceso y autentificación combinados

El acceso a la URL <code>departamentos.iesgn.org/secreto</code> se hace forma directa desde la intranet, desde la red pública te pide la autentificación.

Para que desde la intranet accedamos directamente y desde el exterior nos pida autenticación, modificaremos el archivo de configuración añadiendo las siguientes lineas:

~~~
location /secreto {
  satisfy any;

  allow 192.168.200.0/24;
  deny all;

  auth_basic "Acceso exterior";
  auth_basic_user_file /etc/nginx/.htpasswd;
 }
~~~

Y reiniciamos el servicio:

~~~
sudo systemctl restart nginx
~~~