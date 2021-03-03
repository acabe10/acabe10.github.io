---
layout: post
title: Docker compose
tags: [all, Docker, Docker-compose, Debian]
---
# Introducción

Buenas, en este post vamos a realizar una serie de ejercicios para implantar aplicaciones php en docker. [Aquí](https://github.com/acabe10/docker_build) os dejo el repositorio dónde tengo todos los ficheros.

## Tarea 1: Ejecución de una aplicación web PHP en docker

Objetivos:

* Queremos ejecutar en un contenedor docker la aplicación web escrita en PHP: bookMedik (https://github.com/evilnapsis/bookmedik).

* Es necesario tener un contenedor con mariadb donde vamos a crear la base de datos y los datos de la aplicación. El script para generar la base de datos y los registros lo encuentras en el repositorio y se llama schema.sql. Debes crear un usuario con su contraseña en la base de datos. La base de datos se llama bookmedik y se crea al ejecutar el script.

* Ejecuta el contenedor mariadb y carga los datos del script schema.sql. Para más información.

* El contenedor mariadb debe tener un volumen para guardar la base de datos.

* El contenedor que creas debe tener un volumen para guardar los logs de apache2.

* Crea una imagen docker con la aplicación desde una imagen base de debian o ubuntu. Ten en cuenta que el fichero de configuración de la base de datos (core\controller\Database.php) lo tienes que configurar utilizando las variables de entorno del contenedor mariadb. (Nota: Para obtener las variables de entorno en PHP usar la función getenv. Para más infomación).

* La imagen la tienes que crear en tu máquina con el comando docker build.

* Crea un script con docker compose que levante el escenario con los dos contenedores.(Usuario: admin, contraseña: admin).

Lo primero que tendremos que hacer será clonar el repositorio de la aplicación:

~~~
git clone https://github.com/evilnapsis/bookmedik.git
~~~

Y tendremos creado un repositorio en `Github` con el siguiente contenido, dónde `bookmedik` es el repositorio que acabamos de crear

~~~
├── build
│   ├── bookmedik
│   ├── Dockerfile
│   └── script.sh
├── deploy
│   └── docker-compose.yml
└── README.md
~~~

Primero, para crear el volumen con las tablas que necesitamos, creamos el `docker-compose` con la siguiente información:

~~~
version: '3.1'

db:
     container_name: servidor_mysql
     image: mariadb
     restart: always
     environment:
           MYSQL_DATABASE: bookmedik
           MYSQL_USER: user_bookmedik
           MYSQL_PASSWORD: pass_bookmedik
           MYSQL_ROOT_PASSWORD: asdasd
     volumes:
           - /opt/mysql:/var/lib/mysql
~~~

Lo ejecutamos:

~~~
cd deploy
docker-compose up -d
~~~

Ejecutamos el script para crear las tablas que se encuentra en el repositorio clonado, no sin antes eliminar la línea que dice lo siguiente:

~~~
create database bookmedik
~~~

Ya que hemos creado la base de datos desde el `docker-compose`. Ejecutamos el script:

~~~
cat ../build/bookmedik/schema.sql | docker exec -i servidor_mysql /usr/bin/mysql -u user_bookmedik --password=pass_bookmedik bookmedik
~~~

Creamos un `Dockerfile` en `build` con el siguiente contenido para crear una imagen desde una imagen base de debian, en la cuál instalaremos `apache2` y `php`:

~~~
FROM debian
MAINTAINER Alejandro Cabezas "alejandrocabezab@gmail.com"

RUN apt-get update && apt-get install -y apache2 \
libapache2-mod-php7.3 \
php7.3 \
php7.3-mysql \
&& apt-get clean \
&& rm -rf /var/lib/apt/lists/*

EXPOSE 80

RUN rm /var/www/html/index.html
COPY bookmedik /var/www/html/
ADD script.sh /usr/local/bin/

RUN chmod +x /usr/local/bin/script.sh

ENV DATABASE_USER user_bookmedik
ENV DATABASE_PASSWORD pass_bookmedik
ENV DATABASE_HOST db

CMD ["script.sh"]
~~~

El `script` que se ejecutará tendrá el siguiente contenido:

~~~
#!/bin/bash

sed -i "s/$this->user=\"root\";/$this->user=\"$DATABASE_USER\";/g" /var/www/html/core/controller/Database.php
sed -i "s/$this->pass=\"\";/$this->pass=\"$DATABASE_PASSWORD\";/g" /var/www/html/core/controller/Database.php
sed -i "s/$this->host=\"localhost\";/$this->host=\"$DATABASE_HOST\";/g" /var/www/html/core/controller/Database.php
apache2ctl -D FOREGROUND
~~~

Ejecutamos la instrucción para construir nuestra imagen (dentro del directorio build):

~~~
docker build -t acabe10/bookmedik:v1 .
~~~

Y comprobamos que se haya hecho:

~~~
docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
acabe10/bookmedik             v1                  e71855ce0820        40 seconds ago      251MB
~~~

Ahora volvemos a editar el `docker-compose.yml` y añadimos lo siguiente:

~~~
version: '3.1'

services:

  db:
     container_name: servidor_mysql
     image: mariadb
     restart: always
     environment:
           MYSQL_DATABASE: bookmedik
           MYSQL_USER: user_bookmedik
           MYSQL_PASSWORD: pass_bookmedik
           MYSQL_ROOT_PASSWORD: asdasd
     volumes:
           - /opt/mysql:/var/lib/mysql

  bookmedik:
     container_name: bookmedik
     image: acabe10/bookmedik:v1
     restart: always
     ports:
         - 8080:80
     volumes:
         - /opt/bookmedik_logs:/var/log/apache2
~~~

Lo ejecutamos:

~~~
docker-compose up -d
~~~

Y ya podremos acceder a la web para comprobarlo:

![1](/assets/img/posts/tareas-docker/1.png)

![2](/assets/img/posts/tareas-docker/2.png)

Para subir la imagen:

~~~
docker login
~~~

Y le hacemos el `push`:

~~~
docker push acabe10/bookmedik:v1
~~~

#### Repositorio: https://github.com/acabe10/docker_build/tree/main/tarea1

## Tarea 2: Ejecución de una aplicación web PHP en docker

Objetivos:

* Realiza la imagen docker de la aplicación a partir de la imagen oficial PHP que encuentras en docker hub. Lee la documentación de la imagen para configurar una imagen con `apache2` y `php`, además seguramente tengas que instalar alguna extensión de php.

* Crea esta imagen en docker hub.

* Crea un script con docker compose que levante el escenario con los dos contenedores.

Hemos copiado los directorios de la tarea 1 y hemos hecho los siguientes cambios en el `Dockerfile`, hemos tenido que instalar la extension `mysqli` para poder conectar con la base de datos:

~~~
FROM php:7.2-apache
MAINTAINER Alejandro Cabezas "alejandrocabezab@gmail.com"

EXPOSE 80

RUN docker-php-ext-install mysqli
COPY bookmedik /var/www/html/
ADD script.sh /usr/local/bin/

RUN chmod +x /usr/local/bin/script.sh

ENV DATABASE_USER=user_bookmedik
ENV DATABASE_PASSWORD=pass_bookmedik
ENV DATABASE_HOST=servidor_mysql

CMD ["script.sh"]
~~~ 

Hemos creado la nueva imagen:

~~~
docker build -t acabe10/bookmedik:v2 . 
~~~

Hemos actualizado el `docker-compose.yml` con la nueva versión:

~~~
version: '3.1'

services:

  db:
     container_name: servidor_mysql
     image: mariadb
     restart: always
     environment:
           MYSQL_DATABASE: bookmedik
           MYSQL_USER: user_bookmedik
           MYSQL_PASSWORD: pass_bookmedik
           MYSQL_ROOT_PASSWORD: asdasd
     volumes:
           - /opt/mysql:/var/lib/mysql

  bookmedik:
     container_name: bookmedik
     image: acabe10/bookmedik:v2
     restart: always
     ports:
         - 8080:80
     volumes:
         - /opt/bookmedik_logs:/var/log/apache2
~~~

Y lanzamos el `docker-compose`:

~~~
docker-compose up -d
~~~

Comprobamos:

![3](/assets/img/posts/tareas-docker/3.png)

![4](/assets/img/posts/tareas-docker/4.png)

#### Repositorio: https://github.com/acabe10/docker_build/tree/main/tarea2

## Tarea 3: Ejecución de una aplicación PHP en docker

Objetivos:

* En este caso queremos usar un contenedor que utilice nginx para servir la aplicación PHP. Puedes crear la imagen desde una imagen base debian o ubuntu o desde la imagen oficial de nginx.

* Vamos a crear otro contenedor que sirva php-fpm.

* Y finalmente nuestro contenedor con la aplicación.

* Crea un script con docker compose que levante el escenario con los tres contenedores.

Tenemos el repositorio con la siguiente estructura:

~~~
.
├── build
│   ├── bookmedik
│   ├── default.conf
│   ├── Dockerfile
│   └── script.sh
├── build-php
│   └── Dockerfile
├── deploy
│   └── docker-compose.yml
└── README.md
~~~

Hemos realizado la creación de dos imágenes, la primera desde una imagen oficial de `nginx`, y la segunda desde la imagen oficial de `php-fpm`. No hemos usado una imagen ya creada porque nos hace falta instalar el plugin de `mysqli`.

Muestro el Dockerfile de la imagen de nginx:

~~~
FROM nginx
MAINTAINER Alejandro Cabezas "alejandrocabezab@gmail.com"

EXPOSE 80

RUN rm /etc/nginx/conf.d/default.conf
COPY bookmedik /opt/bookmedik
COPY default.conf /etc/nginx/conf.d/default.conf

ADD script.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/script.sh

ENV DATABASE_USER=user_bookmedik
ENV DATABASE_PASSWORD=pass_bookmedik
ENV DATABASE_HOST=servidor_mysql

CMD ["script.sh"]
~~~

Como vemos, hemos copiado el directorio de bookmedik a `/opt/bookmedik`, para posteriormente en el script copiarlo a `/code`, dónde alojaremos nuestro sitio para que el contenedor de php pueda también obtener los datos a través de un volumen.

Muestro el script, en el cuál hacemos el mismo procedimiento que anteriormente, excepto que copiamos el directorio de bookmedik a `/code` y arrancamos `nginx`:

~~~
#!/bin/bash

cp -R /opt/bookmedik/* /code
sed -i "s/$this->user=\"root\";/$this->user=\"$DATABASE_USER\";/g" /code/core/controller/Database.php
sed -i "s/$this->pass=\"\";/$this->pass=\"$DATABASE_PASSWORD\";/g" /code/core/controller/Database.php
sed -i "s/$this->host=\"localhost\";/$this->host=\"$DATABASE_HOST\";/g" /code/core/controller/Database.php

nginx -g 'daemon off;'
~~~

Muestro también el fichero de configuración de nginx que hace que conecte con el contenedor de `php`:

~~~
server {
    index index.php index.html;
    server_name _;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /code;

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
~~~

Creamos la imagen:

~~~
docker build -t acabe10/bookmedik:v3 .
~~~

Muestro la creación de la imagen para el contenedor php:

~~~
FROM php:7.2-fpm
MAINTAINER Alejandro Cabezas "alejandrocabezab@gmail.com"

RUN docker-php-ext-install mysqli
~~~

Creamos la imagen:

~~~
docker build -t acabe10/php-fpm-mysqli:v1 .
~~~

Muestro el deploy:

~~~
version: '3.1'

services:

  db:
     container_name: servidor_mysql
     image: mariadb
     restart: always
     environment:
           MYSQL_DATABASE: bookmedik
           MYSQL_USER: user_bookmedik
           MYSQL_PASSWORD: pass_bookmedik
           MYSQL_ROOT_PASSWORD: asdasd
     volumes:
           - /opt/mysql:/var/lib/mysql

  bookmedik:
     container_name: bookmedik
     image: acabe10/bookmedik:v3
     restart: always
     ports:
         - 8080:80
     volumes:
         - /opt/bookmedik-php:/code

  php:
     container_name: php
     image: acabe10/php-fpm-mysqli:v1
     restart: always
     volumes:
          - /opt/bookmedik-php:/code
~~~

Para que el servidor `php` pueda obtener los datos del servidor `bookmedik`, tenemos que compartir el volumen, por ello anteriormente tuvimos que copiar el directorio de bookmedik en un directorio diferente.

Si ejecutamos el deploy:

~~~
docker-compose up -d
   Creating bookmedik      ... done
   Creating servidor_mysql ... done
   Creating php            ... done
~~~

Y vamos al navegador, entraremos correctamente:

![16](/assets/img/posts/tareas-docker/16.png)

#### Repositorio: https://github.com/acabe10/docker_build/tree/main/tarea3

## Tarea 4: Ejecución de un CMS en docker

Objetivos:

* A partir de una imagen base (que no sea una imagen con el CMS), genera una imagen que despliegue un CMS PHP (que no sea wordpress).

* Crea los volúmenes necesarios para que la información que se guarda sea persistente.

Vamos a generar una imagen con Drupal desde una imagen base Debian.

Primero descargamos los archivos de Drupal desde su página oficial y creamos la estructura como anteriormente:

~~~
.
├── build
│   ├── Dockerfile
│   ├── drupal
│   └── script.sh
├── deploy
│   └── docker-compose.yml
└── README.md
~~~

El `Dockerfile` para generar la imagen será el siguiente:

~~~
FROM debian
MAINTAINER Alejandro Cabezas "alejandrocabezab@gmail.com"

EXPOSE 80

RUN apt-get update && apt-get install -y apache2 \
libapache2-mod-php \
php \
php-mysql \
php-dom  \
php-gd \
php-simplexml \
php-xml \
php7.3-mbstring \
&& apt-get clean \
&& rm -rf /var/lib/apt/lists/*

RUN rm /var/www/html/index.html
COPY drupal /var/www/html/
COPY drupal/sites /opt/drupal/sites
COPY drupal/profiles /opt/drupal/profiles
COPY drupal/modules /opt/drupal/modules
COPY drupal/themes /opt/drupal/themes

ADD script.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/script.sh

CMD ["script.sh"]
~~~

Hemos tenido que copiar la información por separado por dos motivos:

1. Para poder guardar los datos ya instalados en volúmenes, necesitamos rellenar el volúmen una vez montado, porque si no, perderíamos los datos dentro de él, entonces posteriormente en el `script.sh` haremos un `cp` de esa carpeta.

2. Podríamos haber metido todos los datos en una carpeta para luego hacer un `cp` de cada subdirectorio que queremos, pero Drupal no nos permite hacer la instalación si movemos los ficheros a otro directorio de la carpeta original.

Mostramos el script para copiar los directorios anteriormente copiados en el `Dockerfile` y para crear los directorios necesarios para poder instalar `Drupal`:

~~~
#!/bin/bash

cp -R /opt/drupal/sites /var/www/html/
cp -R /opt/drupal/modules /var/www/html/
cp -R /opt/drupal/themes /var/www/html/
cp -R /opt/drupal/profiles /var/www/html/
mkdir -p /var/www/html/sites/default/files/translations
chmod -R 777 /var/www/html/sites/default
cp /var/www/html/sites/default/default.settings.php /var/www/html/sites/d$
chmod -R 777 /var/www/html/sites/default/settings.php

apache2ctl -D FOREGROUND
~~~

Muestro el `docker-compose`:

~~~
version: '3.1'

services:

  db:
     container_name: servidor_mysql
     image: mariadb
     restart: always
     environment:
           MYSQL_DATABASE: db_drupal
           MYSQL_USER: user_drupal
           MYSQL_PASSWORD: pass_drupal
           MYSQL_ROOT_PASSWORD: root
     volumes:
           - /opt/mysql:/var/lib/mysql

  drupal:
     container_name: drupal
     image: acabe10/drupal:v1
     restart: always
     ports:
         - 8080:80
     volumes:
         - /opt/drupal/sites:/var/www/html/sites/
         - /opt/drupal/themes:/var/www/html/themes/
         - /opt/drupal/profiles:/var/www/html/profiles/
         - /opt/drupal/modules:/var/www/html/modules/
~~~

Y ahora ejecutamos el `deploy`:

~~~
docker-compose up -d
~~~

Empezamos con la instalación de Drupal:

![5](/assets/img/posts/tareas-docker/5.png)

Introducimos los datos que hemos puesto en la máquina `servidor-mysql`:

![6](/assets/img/posts/tareas-docker/6.png)

Comenzará la instalación:

![7](/assets/img/posts/tareas-docker/7.png)

Y una vez finalizada:

![8](/assets/img/posts/tareas-docker/8.png)

Ahora para comprobar que los volúmenes funcionan correctamente:

~~~
docker-compose stop 
     Stopping servidor_mysql ... done
     Stopping drupal         ... done
docker-compose rm  
     Going to remove servidor_mysql, drupal
     Are you sure? [yN] y
     Removing servidor_mysql ... done
     Removing drupal         ... done
docker-compose up -d
     Creating drupal         ... done
     Creating servidor_mysql ... done
~~~

Y si volvemos a meternos y ponemos los datos que nos pide la instalación:

![9](/assets/img/posts/tareas-docker/9.png)

Y si vamos al sitio:

![10](/assets/img/posts/tareas-docker/10.png)

Sólo nos quedaría eliminar el fichero de instalación para que no nos pidiera instalar cada vez que iniciamos la máquina de nuevo después de borrarla.

#### Repositorio: https://github.com/acabe10/docker_build/tree/main/tarea4

## Tarea 5: Ejecución de un CMS en docker

Objetivos:

* Busca una imagen oficial de un CMS PHP en docker hub (distinto al que has instalado en la tarea anterior, ni wordpress), y crea los contenedores necesarios para servir el CMS, siguiendo la documentación de docker hub.

Vamos a instalar `Joomla`, para ello hemos seguido la [documentación oficial de Dockerhub](https://hub.docker.com/_/joomla). Muestro el `docker-compose`:

~~~
version: '3.1'

services:

  joomladb:
    container_name: servidor_mysql
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
~~~

Ejecutamos el deploy:

~~~
docker-compose up -d
~~~

Y accedemos a la página de instalación:

![11](/assets/img/posts/tareas-docker/11.png)

Ponemos los datos que hemos puesto en nuestro servidor mysql:

![12](/assets/img/posts/tareas-docker/12.png)

Vemos una visión general para que todo este correcto:

![13](/assets/img/posts/tareas-docker/13.png)

Y ya tendremos `Joomla` instalado:

![14](/assets/img/posts/tareas-docker/14.png)

Si accedemos al sitio:

![15](/assets/img/posts/tareas-docker/15.png)

En la documentación oficial no nos proporciona información sobre volúmenes para poder hacer los datos persistentes, pero podríamos hacer un volumen del contenedor `mariadb` y de los datos que se acaban de instalar en el contenedor `joomla`, exactamente igual que lo hicimos en el ejercicio anterior.

#### Repositorio: https://github.com/acabe10/docker_build/tree/main/tarea5