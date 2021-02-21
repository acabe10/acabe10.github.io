---
layout: post
title: Cambiar servidor base de datos Drupal
tags: [CMS, PHP, Drupal, Base de datos, BBDD]
---
# Introducción

Buenas, en esta práctica vamos a cambiar el servidor de base de datos que instalamos en la [anterior práctica](https://acabe10.github.io/2021-02-19-Instalaci%C3%B3n-CMS-Drupal/) para así tener los datos de nuestro CMS Drupal en una máquina remota en la misma red.

## Crear copia de base de datos

Primero vamos a crear una copia de nuestra base de datos, vamos a usar <code>mysqldump</code>, que es una herramienta que viene instalada por defecto con <code>MariaDB</code>. Le indicamos el usuario propietario, la contraseña y la base de datos que queremos copiar:

~~~
mysqldump --user=drupal --password=drupal drupal > backup.sql
~~~

## Crear otra máquina y configurarle un servidor de base de datos

Creamos otra máquina en nuestro <code>VagrantFile</code> que se encuentre en una red interna:

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

        config.vm.define :server2 do |server2|
                server2.vm.box = "debian/buster64"
                server2.vm.hostname = "server2"
                server2.vm.network "public_network",:bridge=>"wlp5s0"
                server2.vm.network "private_network", ip: "10.0.0.2",
                        virtualbox__intnet: "intranet"
        end
end
~~~

Desde el primer nodo, enviamos nuestra copia de seguridad al segundo nodo:

~~~
scp backup.sql vagrant@10.0.0.2:/home/vagrant
~~~

Puede que nos dé un fallo de <code>Permission denied (publickey)</code> al estar trabajando con <code>Vagrant</code>, para solucionarlo, editamos la siguiente línea en el fichero <code>/etc/ssh/sshd_config</code>:

~~~
PasswordAuthentication no
~~~

Y la dejamos de la siguiente forma:

~~~
PasswordAuthentication yes
~~~

Ahora nos vamos al segundo nodo. Instalamos <code>MariaDB</code> como lo hicimos en el primer nodo:

~~~
sudo apt install mariadb-server

sudo mysql_secure_installation
~~~

Configuramos dicho servidor de base de datos para que pueda ser accesible desde el exterior:

~~~
nano /etc/mysql/mariadb.conf.d/50-server.cnf
~~~

Descomentamos las siguientes líneas (si están comentadas):

~~~
port                   = 3306
bind-address            = 127.0.0.1
skip-external-locking
~~~

Y cambiamos <code>bin-address</code> para que se pueda conectar desde cualquier IP, o si preferimos, de la IP de la máquina:

~~~
bind-address            = 0.0.0.0
~~~

Reiniciamos mariadb:

~~~
sudo systemctl restart mariadb
~~~

## Creamos usuario para la nueva base de datos y la base de datos que usaremos

Entramos como root:

~~~
sudo mysql -u root -p
~~~

Creamos el usuario:

~~~
GRANT ALL PRIVILEGES ON *.* TO 'externo'@'%' IDENTIFIED BY 'externo' WITH GRANT OPTION;flush privileges;
~~~

Y creamos la base de datos:

~~~
> create database backup;
~~~

## Restauramos base de datos

Restauramos la copia de seguridad en la nueva base de datos creada:

~~~
mysql backup --user=externo --password=externo < backup.sql
~~~

Y si la observamos, veremos que tenemos todas las tablas correctamente:

~~~
MariaDB [backup]> show tables;
+-----------------------------------------+
| Tables_in_backup                        |
+-----------------------------------------+
| batch                                   |
| block_content                           |
| block_content__body                     |
| block_content_field_data                |
| block_content_field_revision            |
| block_content_revision                  |
| block_content_revision__body ..........
...........................
~~~

## Desinstalamos servidor de MariaDB en primer nodo

En el servidor principal, desinstalamos <code>MariaDB</code>:

~~~
sudo apt purge mariadb-server
~~~

Si queda algún tipo de rastro:

~~~
sudo apt autoremove
~~~

Ahora si nos vamos a nuestra página, tendremos un error como este:

![22](/assets/img/posts/drupal-bbdd/22.png)

## Solucionar el problema

Para que Drupal apunte a la base de datos que hemos creado en nuestro nodo secundario, editaremos su fichero de configuración:

~~~
sudo nano /var/www/drupal/sites/default/settings.php
~~~

Y en las últimas líneas, le indicamos dónde se encuentra nuestra base de datos, el usuario que la administra, el nombre de la base de datos y la dirección ip de la máquina:

~~~
$databases['default']['default'] = array (
  'database' => 'backup',
  'username' => 'externo',
  'password' => 'externo',
  'prefix' => '',
  'host' => '10.0.0.2',
  'port' => '3306',
  'namespace' => 'Drupal\\Core\\Database\\Driver\\mysql',
  'driver' => 'mysql',
);
~~~

Reiniciamos el servicio de <code>apache2</code>:

~~~
sudo systemctl restart apache2
~~~

Y volveremos a tener activo nuestro sitio:

![23](/assets/img/posts/drupal-bbdd/23.png)