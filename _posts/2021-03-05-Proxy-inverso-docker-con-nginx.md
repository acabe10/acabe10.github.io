---
layout: post
title: Proxy inverso docker con nginx
tags: [all, proxy, Debian, proxy inverso, docker, docker-compose]
---
# Introducción

Buenas, en este post vamos a instalar `joomla` y `nextcloud` usando un `docker-compose`, y posteriormente configuraremos un proxy inverso en nuestra máquina con nginx para que podamos acceder a las dos aplicaciones mediante un nombre y no mediante un puerto.

## Instalación

Haremos la instalación de `joomla` y `nextcloud`, para ello tenemos el siguiente `docker-compose`:

~~~
version: '3.1'

services:

  joomladb:
    container_name: joomladb
    image: mariadb
    restart: always
    environment:
        MYSQL_ROOT_PASSWORD: root
        MYSQL_DATABASE: db_joomla
        MYSQL_USER: user_joomla
        MYSQL_PASSWORD: pass_joomla

  joomla:
    container_name: joomla
    image: joomla
    restart: always
    ports:
        - 8080:80
    environment:
        JOOMLA_DB_HOST: joomladb
        JOOMLA_DB_USER: user_joomla
        JOOMLA_DB_PASSWORD: pass_joomla
        JOOMLA_DB_NAME: db_joomla

  nextcloud_db:
    container_name: nextcloud_db
    image: mariadb
    restart: always
    environment:
        MYSQL_DATABASE: db_nextcloud
        MYSQL_USER: user_nextcloud
        MYSQL_PASSWORD: pass_netxcloud
        MYSQL_ROOT_PASSWORD: root

  nextcloud:
    container_name: nextcloud
    image: nextcloud
    restart: always
    environment:
        MYSQL_HOST: nextcloud_db
        MYSQL_USER: user_nextcloud
        MYSQL_PASSWORD: pass_netxcloud
        MYSQL_DATABASE: db_nextcloud
    ports:
        - 8081:80

~~~

Desplegamos:

~~~
docker-compose up -d
~~~

## Editando nginx

Suponiendo que ya tenemos instalado nginx en nuestro equipo, vamos a hacer dos virtualhost, para la primera aplicación:

~~~
sudo nano /etc/nginx/sites-available/app1
~~~

Y añadimos:

~~~
server {
        listen 80;
        listen [::]:80;

        index index.html index.htm index.nginx-debian.html;

        server_name www.app1.org;

        location / {
                proxy_pass http://localhost:8080;
        }
}
~~~

Y para la segunda:

~~~
sudo nano /etc/nginx/sites-available/app2
~~~

Y añadimos:

~~~
server {
        listen 80;
        listen [::]:80;

        index index.html index.htm index.nginx-debian.html;

        server_name www.app2.org;

        location / {
                proxy_pass http://localhost:8081;
        }
}
~~~

Habilitamos los dos sitios:

~~~
sudo ln -s /etc/nginx/sites-available/app1 /etc/nginx/sites-enabled/app1
sudo ln -s /etc/nginx/sites-available/app2 /etc/nginx/sites-enabled/app2
~~~

Reiniciamos nginx:

~~~
sudo systemctl restart nginx
~~~

Añadimos a nuestro fichero:

~~~
sudo nano /etc/hosts
~~~

El siguiente contenido:

~~~
127.0.0.1       www.app1.org
127.0.0.1       www.app2.org
~~~

Y comprobamos:

![9](/assets/img/posts/squid/9.png)

![10](/assets/img/posts/squid/10.png)