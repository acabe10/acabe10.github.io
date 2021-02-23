---
layout: post
title: Aumento de rendimiento de servidores web con Varnish
tags: [all, Varnish, proxy, nginx]
---
# Introducción

Buenas, en este post vamos a comparar el rendimiento de distintas configuraciones de servidores web sirviendo páginas dinámicas programadas con PHP, en concreto vamos a servir un CMS Wordpress.

Las configuraciones que vamos a realizar son las siguientes:

- Módulo php5-apache2
- PHP-FPM (socket unix) + apache2
- PHP-FPM (socket TCP) + apache2
- PHP-FPM (socket unix) + nginx
- PHP-FPM (socket TCP) + nginx

Para cada una de las configuraciones hemos hecho pruebas de rendimiento con el comando `ab`, por ejemplo durante 10 segundos , hemos hecho peticiones con 200 concurrentes:

~~~
ab -t 10 -c 200 -k http://172.22.x.x/wordpress/index.php
~~~

Después de hacer muchas pruebas de rendimiento con un número variable de peticiones concurrentes (1, 10, 25, 50, 75, 100, 250, 500, 1000) y distintas direcciones del wordpress. Los resultados obtenidos son los siguientes:

![php1](/assets/img/posts/varnish/php1.png)

Podemos determinar que la opción que nos ofrece más rendimiento es `nginx + fpm_php (socket unix)`. Cuyo resultado es aproximadamente unas 600 peticiones/segundo (parámetro Requests per second de ab).

A partir de esa configuración vamos a intentar aumentar el rendimiento de nuestro servidor.

# Preparamos máquina

Hemos ejecutado una instancia en OpenStack y le hemos cambiado el fichero `host` en nuestra receta de `ansible` ya configurada para que instale por defecto la [configuración elegida](https://github.com/josedom24/ansible_nginx_fpm_php):

~~~
nano host

[servidores_web]
nodo1 ansible_ssh_host=172.22.201.20 ansible_python_interpreter=/usr/bin/python3
~~~

Ejecutamos la receta:

~~~
ansible-playbook site.yaml
~~~

Y accedemos y terminamos de instalar WordPress:

![1](/assets/img/posts/varnish/1.png)

![2](/assets/img/posts/varnish/2.png)

# Pruebas de rendimiento con comando ab

Instalamos primero el comando `ab`:

~~~
sudo apt install apache2-utils
~~~

Después de cada prueba ejecutaremos la siguiente instrucción:

~~~
sudo systemctl restart nginx
sudo systemctl restart php7.3-fpm.service
~~~

## 50 peticiones concurrentes

~~~
ab -t 10 -c 50 -k http://127.0.0.1/wordpress/index.php
~~~

~~~
...
Requests per second:    31.79 [#/sec] (mean)
...
~~~

## 100 peticiones concurrentes

~~~
ab -t 10 -c 100 -k http://127.0.0.1/wordpress/index.php
~~~

~~~
...
Requests per second:    28.87 [#/sec] (mean)
...
~~~

## 250 peticiones concurrentes

~~~
ab -t 10 -c 250 -k http://127.0.0.1/wordpress/index.php
~~~

~~~
...
Requests per second:    2663.01 [#/sec] (mean)
...
~~~

## 500 peticiones concurrentes

~~~
ab -t 10 -c 500 -k http://127.0.0.1/wordpress/index.php
~~~

~~~
...
Requests per second:    3455.01 [#/sec] (mean)
...
~~~

# Instalación y configuración varnish

Instalamos varnish:

~~~
sudo apt install varnish
~~~

Abrimos el archivo de configuración:

~~~
sudo nano /etc/default/varnish
~~~

Y editamos el demonio para que escuche las peticiones que llegan al puerto 80 desde el exterior:

~~~
DAEMON_OPTS="-a :80 \
             -T localhost:6082 \
             -f /etc/varnish/default.vcl \
             -S /etc/varnish/secret \
             -s malloc,256m"
~~~

También tendremos que hacer que `Varnish` arranque en el puerto 80, para ello editamos la unidad de `systemd`:

~~~
sudo nano /lib/systemd/system/varnish.service
~~~

Y cambiamos la siguiente línea:

~~~
ExecStart=/usr/sbin/varnishd -j unix,user=vcache -F -a :80 -T localhost:6082 -f /etc/varnish/default.vcl -S /etc/varnish/secret -s malloc,256m
~~~

Ya que hemos configurado `varnish` para que escuche en el puerto 80, esté tendrá que enviar las peticiones al servidor web, el cuál lo vamos a poner en el puerto 8080, para ello, primero editamos el fichero de `varnish` que dice dónde estará el servidor web:

~~~
sudo nano /etc/varnish/default.vcl
~~~

Y editamos:

~~~
backend default {
    .host = "127.0.0.1";
    .port = "8080";
}
~~~

# Configuración nginx

Vamos a configurar a nginx para que escuche en el puerto 8080, para ello abrimos el fichero de configuración:

~~~
sudo nano /etc/nginx/sites-available/default
~~~

Y editamos las siguientes líneas:

~~~
server {
        listen 8080 default_server;
        listen [::]:8080 default_server;
~~~

Podemos comprobar efectivamente que se está ejecutando el wordpress correctamente:

![3](/assets/img/posts/varnish/3.png)

## Comprobaciones

Reiniciamos los servicios:

~~~
sudo systemctl daemon-reload
sudo systemctl restart varnish
sudo systemctl restart nginx
~~~

Y hacemos las mismas pruebas que antes acordándonos de reiniciar los servicios de `nginx` y `php` como hicimos anteriormente:

### 50 peticiones concurrentes

~~~
ab -t 10 -c 50 -k http://127.0.0.1/wordpress/index.php
~~~

~~~
...
Requests per second:    9357.43 [#/sec] (mean)
...
~~~

### 100 peticiones concurrentes

~~~
ab -t 10 -c 100 -k http://127.0.0.1/wordpress/index.php
~~~

~~~
...
Requests per second:    10991.40 [#/sec] (mean)
...
~~~

### 250 peticiones concurrentes

~~~
ab -t 10 -c 250 -k http://127.0.0.1/wordpress/index.php
~~~

~~~
...
Requests per second:    11608.69 [#/sec] (mean)
...
~~~

### 500 peticiones concurrentes

~~~
ab -t 10 -c 500 -k http://127.0.0.1/wordpress/index.php
~~~

~~~
...
Requests per second:    11208.79 [#/sec] (mean)
...
~~~

Si hacemos varias peticiones al servidor, sólo se mostrará una, ya que _varnish_ hace también de servidor caché:

~~~
less /var/log/nginx/access.log

127.0.0.1 - - [13/Feb/2021:09:28:06 +0000] "GET /wordpress HTTP/1.0" 301 185 "-" "ApacheBench/2.3"
~~~