---
layout: post
title: Instalación servidor LEMP en OVH
tags: [all, Nginx, Servidor web, Virtualhosting, LEMP, PHP-FPM, OVH]
---
# Introducción

Buenas, en este post vamos a configurar un servidor en OVH, el cuál nos han cedido nuestros profesores del IES Gonzalo Nazareno para poder prácticar durante este año, seguramente que haga más tareas sobre este servidor, el cuál encontraremos en la etiqueta del post como <code>OVH</code>.

Vamos a realizar los siguientes puntos:

1. Instala un servidor web nginx
2. Instala un servidor de base de datos MariaDB. Ejecuta el programa necesario para asegurar el servicio, ya que lo vamos a tener corriendo en el entorno de producción.
3. Instala un servidor de aplicaciones php php-fpm.
4. Crea un virtualhost al que vamos acceder con el nombre <code>www.iesgnXX.es</code>.
5. Cuando se acceda al virtualhost por defecto default nos tiene que redirigir al virtualhost que hemos creado en el punto anterior.
6. Cuando se acceda a <code>www.iesgnXX.es</code> se nos redigirá a la página <code>www.iesgnXX.es/principal</code>
7. En la página <code>www.iesgnXX.es/principal</code> se debe mostrar una página web estática (utiliza alguna plantilla para que tenga hoja de estilo).
8. Configura el nuevo virtualhost, para que pueda ejecutar PHP. Determina que configuración tiene por defecto php-fpm (socket unix o socket TCP) para configurar nginx de forma adecuada.
9. Crea un fichero <code>info.php</code> que demuestre que está funcionando el servidor LEMP.

## Ejercicio 1

Ejecutamos lo siguiente para instalar _nginx_:

~~~
sudo apt update && apt -y upgrade && apt install nginx
~~~

## Ejercicio 2

Para instalar <code>MariaDB</code>:

~~~
sudo apt install -y mariadb-server
~~~

Hacemos la instalación segura:

~~~
sudo mysql_secure_installation
~~~

## Ejercicio 3

Instalamos <code>php</code>, <code>php-mysql</code> y <code>php-fpm</code>, que es la implementación alternativa de <code>PHP FastCGI</code>, que junto a <code>nginx</code> tiene un rendimiento óptimo al dividir los servicios:

~~~
sudo apt install php php-fpm php-mysql
~~~

## Ejercicio 4

Creamos la carpeta donde vamos a alojar el VirtualHost:

~~~
sudo mkdir /var/www/iesgn02
~~~

Copiamos el fichero de configuración del VirtualHost por defecto y lo editamos:

~~~
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/iesgn02
sudo nano /etc/nginx/sites-available/iesgn02
~~~

Le dejamos el siguiente contenido ya modificado:

~~~
server {
        listen 80;
        listen [::]:80;

        root /var/www/iesgn02;

        index index.html index.htm index.nginx-debian.html;

        server_name www.iesgn02.es iesgn02.es;

        location / {
                try_files $uri $uri/ =404;
        }
}
~~~

Añadimos el enlace simbólico para que funcione el sitio:

~~~
ln -s /etc/nginx/sites-available/iesgn02 /etc/nginx/sites-enabled/iesgn02
~~~

Y cambiamos el propietario de la carpeta <code>iesgn02</code>

~~~
sudo chown -R www-data. /var/www/
~~~

Ya sólo nos quedaría añadir el registro CNAME en nuestro servicio DNS de OVH que tuviera el siguiente aspecto:

~~~
www.iesgn02.es.        0        CNAME        ned
~~~

Reiniciamos nginx:

~~~
sudo systemctl restart nginx
~~~

## Tarea 5

Tendremos que modificar el VirtualHost por defecto:

~~~
sudo nano /etc/nginx/sites-available/default
~~~

Y añadirle la siguiente línea:

~~~
return 301 $scheme://www.iesgn02.es$request_uri
~~~

## Tarea 6

Creamos el directorio principal:

~~~
sudo mkdir /var/www/iesgn02/principal
~~~

Cambiamos permisos:

~~~
sudo chown -R www-data. /var/www/
~~~

Editamos el fichero de configuración del VirtualHost:

~~~
sudo nano /etc/nginx/sites-available/iesgn02
~~~

Y añadimos la siguiente línea:

~~~
rewrite ^/$ principal/ permanent;
~~~

## Tarea 7

Hemos pasado la plantilla con scp desde nuestro equipo. Ha quedado de la siguiente forma:

![1](/assets/img/posts/lemp-ovh/1.png)

## Tarea 8

Para comprobar que socket está usando PHP:

~~~
cat /etc/php/7.3/fpm/pool.d/www.conf  | egrep 'listen ='

listen = /run/php/php7.3-fpm.sock
~~~

Trae por defecto <code>socket unix</code>, con el que lo haremos nosotros ya que es el recomendado para cuando <code>nginx</code> y <code>php</code> se van a ejecutar en el mismo servidor. Para ello tendremos que editar el fichero de configuración de nuestro VirtualHost:

~~~
nano /etc/nginx/sites-available/iesgn02
~~~

Y añadir las siguientes líneas:

~~~
location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php7.3-fpm.sock;
}
~~~

Tenemos que confirmar que los permisos y usuario por defecto están correctamente en el fichero <code>www.conf</code>:

~~~
cat /etc/php/7.3/fpm/pool.d/www.conf | egrep 'listen\.'

;listen.backlog = 511
listen.owner = www-data
listen.group = www-data
;listen.mode = 0660
; When set, listen.owner and listen.group are ignored
;listen.acl_users =
;listen.acl_groups =
;listen.allowed_clients = 127.0.0.1
~~~

## Tarea 9

Creamos el fichero para probar <code>php</code>:

~~~
echo "<?php phpinfo(); ?>" > /var/www/iesgn02/principal/info.php
~~~

Reiniciamos el servicio:

~~~
sudo systemctl restart nginx
~~~

Y comprobamos:

![2](/assets/img/posts/lemp-ovh/2.png)