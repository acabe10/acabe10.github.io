---
layout: post
title: Instalación CMS Wordpress
tags: [all, CMS, PHP, Wordpress, BBDD remota]
---
# Introducción

Buenas, en esta práctica vamos a instalar el CMS Wordpress siguiendo la [práctica que hicimos anteriormente](https://acabe10.github.io/2021-02-21-Cambiar-servidor-base-de-datos-Drupal/).

## Creamos bbdd en nodo secundario

Lo primero que haremos será crear una base de datos en nuestro nodo secundario, ya que hemos desinstalado MariaDB del principal:

~~~
mysql -u externo -p

> create database wordpress;
~~~

## Configuración de sitio

Creamos un nuevo fichero en <code>/etc/apache2/sites-available</code>:

~~~
sudo nano wordpress.conf
~~~

Con el siguiente contenido:

~~~
<VirtualHost *:80>
        ServerName www.alejandro-wordpress.org
        DocumentRoot /var/www/wordpress
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        <Directory /var/www/wordpress>
                AllowOverride All
        </Directory>
</VirtualHost>
~~~

Nos vamos al directorio dónde vamos a instalar Wordpress:

~~~
cd /var/www
~~~

Descargamos Wordpress copiando el enlace de su página oficial:

~~~
sudo wget https://es.wordpress.org/latest-es_ES.zip
~~~

Lo descomprimimos igual que hicimos con Drupal y limpiamos:

~~~
sudo unzip latest-es_ES.zip

sudo rm latest-es_ES.zip
~~~

Activamos el sitio virtual:

~~~
cd /etc/apache2/

sudo a2ensite wordpress.conf
~~~

Y reiniciamos <code>apache2</code>:

~~~
sudo systemctl restart apache2
~~~

Añadimos la ip a nuestro fichero <code>/etc/hosts</code> igual que hicimos con Drupal:

~~~
192.168.1.175 www.alejandro-wordpress.org
~~~

E ingresamos en la URL, dándonos cuenta de que algo ha fallado:

![24](/assets/img/posts/wordpress/24.png)

No nos hemos dado cuenta que tenemos que decir a <code>Wordpress</code> que la base de datos no está en nuestra máquina anfitriona, para ello, vamos a editar el siguiente fichero:

~~~
cd /var/www/wordpress/

sudo cp wp-config-sample.php wp-config.php

sudo nano wp-config.php
~~~

Wordpress trae una serie de opciones por defecto en un fichero de ejemplo, nosotros pondremos las que nos hacen falta para que apunte al segundo nodo:

~~~
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'externo' );

/** MySQL database password */
define( 'DB_PASSWORD', 'externo' );

/** MySQL hostname */
define( 'DB_HOST', '10.0.0.2' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );
~~~

Una vez realizado, veremos que podemos continuar:

![25](/assets/img/posts/wordpress/25.png)

Hemos ingresado los datos requeridos y hemos instalado <code>Wordpress</code> fácilmente:

![26](/assets/img/posts/wordpress/26.png)

Una vez instalado nos pedirá el acceso:

![27](/assets/img/posts/wordpress/27.png)

Y nos mostrará la página principal de <code>Wordpress</code>:

![28](/assets/img/posts/wordpress/28.png)

Y con esto tendremos finalizada la instalación de otro CMS en nuestro servidor. Podemos comprobar las tablas en nuestro segundo nodo:

~~~
MariaDB [wordpress]> show tables;
+-----------------------+
| Tables_in_wordpress   |
+-----------------------+
| wp_commentmeta        |
| wp_comments           |
| wp_links              |
| wp_options            |
| wp_postmeta           |
| wp_posts              |
| wp_term_relationships |
| wp_term_taxonomy      |
| wp_termmeta           |
| wp_terms              |
| wp_usermeta           |
| wp_users              |
+-----------------------+
12 rows in set (0.001 sec)
~~~