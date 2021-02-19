---
layout: post
title: Instalación CMS Drupal
tags: [CMS, PHP, LAMP, Drupal]
---
# Introducción

Buenas, en esta práctica vamos a instalar una pila LAMP en nuestro servidor local para posteriormente instalar Drupal en él.

## Instalación de un servidor LAMP

Servidor LAMP significa que nuestro servidor tendrá Apache2, MySQL, PHP y estará en una distribución Linux.

Creamos instancia Debian en vagrant con un servidor:

~~~
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
        config.vm.define :server do |server|
                server.vm.box = "debian/buster64"
                server.vm.hostname = "server"
                server.vm.network "public_network",:bridge=>"wlp5s0"
                server.vm.network "private_network", ip: "10.0.0.1",
                        virtualbox__intnet: "intranet"
        end
end
~~~

### Instalación de MariaDB

Primero actualizamos el equipo:

~~~
sudo apt update && sudo apt -y upgrade
~~~

Instalamos MariaDB, que será nuestro servicio MySQL:

~~~
sudo apt install -y mariadb-server
~~~

Hacemos una instalación segura de MariaDB:

~~~
sudo mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

You already have a root password set, so you can safely answer 'n'.

Change the root password? [Y/n] y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
~~~

### Instalación de Apache2

Vamos a instalar también el módulo de PHP para Apache2:

~~~
# apt install -y apache2 libapache2-mod-php
~~~

### Instalación de PHP

Instalamos PHP y su complemento para MySQL:

~~~
# apt install php php-mysql 
~~~

Activamos el módulo de PHP para Apache2:

~~~
# a2enmod php7.3
~~~

Y con esto tendremos nuestro servidor LAMP.

## Instalación de Drupal en mi servidor local

### Configuración de sitio virtual

Configuramos el VirtualHost en Apache2 para poder instalar Drupal. Vamos a crear un nuevo archivo en <code>sites-available</code> con el siguiente contenido: 

~~~
cd /etc/apache2/sites-available/
nano drupal.conf 
~~~

~~~
<VirtualHost *:80>
        ServerName www.alejandro-drupal.org
        DocumentRoot /var/www/drupal
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
~~~

Desactivamos el sitio por defecto:

~~~
a2dissite 000-default.conf
~~~

Recordamos que tenemos que tener todo nuestro contenido de <code>/var/www/</code> y de <code>/etc/apache2/</code> de usuario propietario <code>www-data</code>:

~~~
chown -R www-data:www-data /var/www/
chown -R www-data:www-data /etc/apache2/
~~~

### Creamos usuario MariaDB

Accedemos a MariaDB con el usuario root:

~~~
# mysql -u root -p
~~~

Y creamos el usuario _drupal_ dándole todos los privilegios, también podríamos darle permisos sólo para la base de datos que vamos a crear.

~~~
> GRANT ALL PRIVILEGES ON *.* TO 'drupal'@'%' IDENTIFIED BY 'drupal' WITH GRANT OPTION;flush privileges;
~~~

Creamos la base de datos que vamos a usar:

~~~
> create database drupal;
~~~

### Descarga de versión de Drupal

Descargamos la última versión de Drupal de su página oficial en el directorio <code>/var/www</code>:

~~~
cd /var/www/
wget https://www.drupal.org/download-latest/zip
~~~

Instalamos _unzip_ para poder descomprimir el archivo:

~~~
apt install unzip
~~~

Descomprimimos el archivo y limpiamos el nombre de la carpeta:

~~~
unzip zip
mv drupal-9.0.7/ drupal
~~~

Activamos el sitio ahora que tenemos nuestra carpeta con sus archivos:

~~~
cd /etc/apache2/sites-available/
a2ensite drupal.conf
~~~

Y reiniciamos _apache2_:

~~~
systemctl restart apache2
~~~

### Instalación de Drupal

Metemos la dirección proporcionada en nuestro <code>/etc/hosts</code>:

~~~
192.168.1.175	www.alejandro-drupal.org
~~~

Y nos vamos al navegador con nuestra dirección:

![1](/assets/img/posts/drupal/1.png)

Vamos a explicar los errores que pueden ir saliendonos. El primero de ellos es que el directorio <code>translations</code> no se ha creado por defecto:

![2](/assets/img/posts/drupal/2.png)

Para solucionar el problema:

~~~
mkdir -p /var/www/drupal/sites/default/files/translations
~~~

Y cambiamos los permisos:

~~~
chown -R www-data:www-data /var/www/
~~~

Seguimos seleccionando un perfil de instalación:

![3](/assets/img/posts/drupal/3.png)

El siguiente error que puede salirnos es que nos falten las extensiones PHP requeridas, las URL limpias que no estén activas y que nos haga falta la extensión para la biblioteca unicode:

![4](/assets/img/posts/drupal/4.png)

Para solucionar las extensiones de PHP, basta con instalar las extensiones que nos pide:

~~~
apt install php-dom php-gd php-simplexml php-xml
~~~

Para las URL limpias, tendremos que entrar en la configuración de nuestro VirtualHost:

~~~
nano /etc/apache2/sites-available/drupal.conf
~~~

Y añadir el siguiente directory:

~~~
<Directory /var/www/drupal>
        AllowOverride All
</Directory>
~~~

Después de eso, activamos el módulo <code>rewrite</code> y el fichero <code>.htaccess</code> hará el resto:

~~~
a2enmod rewrite
~~~

Para la biblioteca unicode instalamos la siguiente extensión:

~~~
apt install php7.3-mbstring
~~~

Reiniciamos el servicio de <code>apache2</code> y todos los errores habrán desaparecido:

~~~
systemctl restart apache2
~~~

Ahora seleccionamos la base de datos, el usuario de la base y la contraseña para continuar:

![5](/assets/img/posts/drupal/5.png)

Empezará a instalarse Drupal:

![6](/assets/img/posts/drupal/6.png)

![8](/assets/img/posts/drupal/8.png)

Nos pedirá que configuremos el sitio con un nombre y varias cosas más:

![7](/assets/img/posts/drupal/7.png)

Y terminará de hacer la instalación:

![9](/assets/img/posts/drupal/9.png)

Al finalizar nos mostrará la pantalla principal de Drupal:

![10](/assets/img/posts/drupal/10.png)

## Cambio de tema y instalación de módulo

Para cambiar el tema nos iremos al panel de <code>Administrar > Apariencia > Instalar nuevo tema</code>:

![12](/assets/img/posts/drupal/12.png)

Para obtener un nuevo tema nos vamos a la página oficial de Drupal dónde se encuentran estos:

~~~
https://www.drupal.org/project/project_theme
~~~

Y buscamos uno de nuestra versión:

![13](/assets/img/posts/drupal/13.png)

Seleccionamos instalar mediante URL y se nos instalará el tema deseado:

![14](/assets/img/posts/drupal/14.png)

Ahora nos tendremos que ir a la misma pestaá de <code>Apariencia</code> y pulsar en <code>Instalar y seleccionar de modo predeterminado</code> el tema que hemos instalado:

![15](/assets/img/posts/drupal/15.png)

Y con esto tendremos un sitio totalmente diferente:

![16](/assets/img/posts/drupal/16.png)

Para instalar un módulo nos iremos a la pestaña de <code>Administrar > Ampliar > Instalar nuevo módulo</code> y buscaremos igual que el tema el que queramos:

![17](/assets/img/posts/drupal/17.png)

En nuestro caso hemos instalado un módulo que nos permite crear foros de debate:

![19](/assets/img/posts/drupal/19.png)

Para probar nuestro sitio, hemos creado algún contenido:

![18](/assets/img/posts/drupal/18.png)