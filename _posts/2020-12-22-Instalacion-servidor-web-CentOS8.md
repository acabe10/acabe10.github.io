---
layout: post
title: Instalación y configuración servidor web CentOS 8
tags: [all, OpenStack, Toboso, Servidor web, CentOS8, CentOS]
---
# Introducción

Buenas, en este post vamos a instalar un servidor web en nuestro servidor `quijote`, que tenemos alojado en nuestro escenario de OpenStack, para que sea capaz de ejecutar código php.

## Configuración en quijote

Instalamos el paquete que contiene apache2 en CentOS 8:

~~~
sudo dnf install httpd
~~~

Para poder suministrar las solicitudes a través de HTTP en centos, existe el paquete `firewalld`, que se encarga de administrar los puertos que ofrece, para listar los servicios activos:

~~~
sudo firewall-cmd --permanent --list-all
public
  target: default
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: dhcpv6-client ssh
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules:
~~~

Vamos a añadir el necesario para que podamos acceder a la página que ofrece:

~~~
sudo firewall-cmd --permanent --add-service=http
~~~

Recargamos el servicio:

~~~
sudo systemctl reload httpd
~~~

Creamos una página de prueba:

~~~
sudo su -
echo "<h1>Pagina en Centos 8</h1>" >> /var/www/html/index.html
~~~

## Configuración en dulcinea

Para que el tráfico que llegue por el puerto 80 y 443 se redirija a quijote, añadimos las siguientes reglas a iptables:

~~~
iptables -t nat -A PREROUTING -i eth2 -p tcp --dport 80 -j DNAT --to 10.0.2.4:80
iptables -t nat -A PREROUTING -i eth2 -p tcp --dport 443 -j DNAT --to 10.0.2.4:443
~~~

## Configuración en freston

Añadimos en el fichero:

~~~
sudo nano /var/cache/bind/db.externa.cabezas.gonzalonazareno.org
~~~

La siguiente línea:

~~~
www             IN      CNAME   dulcinea
~~~

Añadimos en el fichero:

~~~
sudo nano /var/cache/bind/db.interna.cabezas.gonzalonazareno.org
~~~

La siguiente línea:

~~~
www             IN      CNAME   quijote
~~~

Y añadimos en el siguiente fichero:

~~~
sudo nano /var/cache/bind/db.dmz.cabezas.gonzalonazareno.org
~~~

La siguiente línea:

~~~
www             IN      CNAME   quijote
~~~

Reiniciamos el servicio:

~~~
sudo systemctl restart bind9
~~~

## Comprobamos

![1](/assets/img/posts/web-centos/1.png)

## Instalación de php

Desinstalamos el módulo de `php` que viene por defecto en Centos 8:

~~~
sudo dnf module disable -y php
~~~

Para instalar php:

~~~
sudo dnf install php
~~~

Iniciamos el servicio y lo habilitamos para que se arranque al inicio:

~~~
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
~~~

Actualizamos el servicio de `httpd`:

~~~
sudo systemctl reload httpd
~~~

Creamos el fichero `php.info`:

~~~
sudo su -
echo "<?php phpinfo();" >> /var/www/html/info.php
~~~

Y comprobamos:

![2](/assets/img/posts/web-centos/2.png)