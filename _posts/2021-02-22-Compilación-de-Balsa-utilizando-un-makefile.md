---
layout: post
title: Compilación de Balsa utilizando un Makefile
tags: [C, Programa, Makefile, Compilación, Balsa]
---
# Introducción

Buenas, en este post vamos a elegir un programa escrito en C y realizaremos los pasos necesarios para compilarlo e instalarlo en nuestro equipo.

## Descarga

Para compilar un programa en C, primero tendremos que descargar los paquetes fuente de dicho programa, nosotros vamos a descargar <code>balsa</code>, que es un cliente de correo bastante robusto para GNOME.

Para su descarga, nos iremos al siguiente link:

https://www.debian.org/distrib/packages

Buscamos por su nombre y <code>Nombres de paquetes fuente</code>:

![1](/assets/img/posts/balsa/1.png)

Seleccionamos el paquete deseado:

![2](/assets/img/posts/balsa/2.png)

Y al final de la página tenemos varias opciones, la que queremos es la de mayor tamaño ya que es la que tiene los paquetes fuente, en nuestro caso, hemos creado una máquina en <code>Vagrant</code> para instalarlo, por ello, copiaremos la dirección de descarga y en nuestra máquina lo descargaremos con <code>wget</code>:

~~~
wget http://deb.debian.org/debian/pool/main/b/balsa/balsa_2.5.6.orig.tar.bz2
~~~

Descomprimimos dicho paquete:

~~~
sudo tar -xjvf balsa_2.5.6.orig.tar.bz2
~~~

Y obtendremos la carpeta con todos los paquetes descomprimidos:

~~~
root@compi:/home/vagrant/balsa-2.5.6# ls -l
total 1796
-rw-rw-r-- 1 vagrant vagrant    609 Feb 25  2018 acinclude.m4
-rw-rw-r-- 1 vagrant vagrant  75611 Jun  1  2018 aclocal.m4
-rw-rw-r-- 1 vagrant vagrant   3659 Feb 25  2018 AUTHORS
-rw-rw-r-- 1 vagrant vagrant   6410 Feb 25  2018 balsa.1.in
-rw-rw-r-- 1 vagrant vagrant   7759 Jun  1  2018 balsa.appdata.xml
-rw-rw-r-- 1 vagrant vagrant    972 Feb 25  2018 balsa.appdata.xml.in
-rw-rw-r-- 1 vagrant vagrant    273 Jun  1  2018 balsa.desktop.in
-rw-rw-r-- 1 vagrant vagrant    283 Feb 25  2018 balsa.desktop.in.in
-rw-rw-r-- 1 vagrant vagrant   3533 Feb 25  2018 balsa-mail.lang
-rw-rw-r-- 1 vagrant vagrant   1511 Feb 25  2018 balsa-mail-style.xml
-rw-rw-r-- 1 vagrant vagrant    297 Jun  1  2018 balsa-mailto-handler.desktop.in
-rw-rw-r-- 1 vagrant vagrant    307 Feb 25  2018 balsa-mailto-handler.desktop.in.in
-rw-rw-r-- 1 vagrant vagrant   4674 Jun  1  2018 balsa.spec
-rw-rw-r-- 1 vagrant vagrant   4686 Feb 25  2018 balsa.spec.in
-rwxrwxr-x 1 vagrant vagrant    580 Feb 25  2018 bootstrap.sh
-rw-rw-r-- 1 vagrant vagrant 350332 Jun  1  2018 ChangeLog
-rwxr-xr-x 1 vagrant vagrant   7333 Jun  1  2018 compile
-rwxr-xr-x 1 vagrant vagrant  43499 Oct 24  2017 config.guess
-rw-rw-r-- 1 vagrant vagrant   5900 Jun  1  2018 config.h.in
-rwxr-xr-x 1 vagrant vagrant  36144 Oct 24  2017 config.sub
-rwxrwxr-x 1 vagrant vagrant 654880 Jun  1  2018 configure
-rw-rw-r-- 1 vagrant vagrant  28021 Jun  1  2018 configure.ac
-rw-rw-r-- 1 vagrant vagrant  35147 Jun  1  2018 COPYING
-rwxr-xr-x 1 vagrant vagrant  23566 Jun  1  2018 depcomp
drwxrwxr-x 9 vagrant vagrant   4096 Jun  1  2018 doc
drwxrwxr-x 2 vagrant vagrant   4096 Jun  1  2018 docs
-rw-rw-r-- 1 vagrant vagrant   3222 Feb 25  2018 gnome-balsa2.png
-rw-rw-r-- 1 vagrant vagrant   3420 Feb 25  2018 HACKING
drwxrwxr-x 5 vagrant vagrant   4096 Jun  1  2018 images
-rw-rw-r-- 1 vagrant vagrant   2705 Feb 25  2018 INSTALL
-rwxr-xr-x 1 vagrant vagrant  15155 Oct 24  2017 install-sh
-rw-rw-r-- 1 vagrant vagrant      0 Jun  1  2018 intltool-extract.in
-rw-rw-r-- 1 vagrant vagrant      0 Jun  1  2018 intltool-merge.in
-rw-rw-r-- 1 vagrant vagrant      0 Jun  1  2018 intltool-update.in
drwxrwxr-x 3 vagrant vagrant   4096 Jun  1  2018 libbalsa
drwxrwxr-x 2 vagrant vagrant   4096 Jun  1  2018 libinit_balsa
drwxrwxr-x 3 vagrant vagrant   4096 Jun  1  2018 libnetclient
-rw-r--r-- 1 vagrant vagrant 324404 Feb  7  2016 ltmain.sh
drwxrwxr-x 2 vagrant vagrant   4096 Jun  1  2018 m4
-rw-rw-r-- 1 vagrant vagrant   2689 Feb 27  2018 Makefile.am
-rw-rw-r-- 1 vagrant vagrant  38281 Jun  1  2018 Makefile.in
-rwxr-xr-x 1 vagrant vagrant   6872 Jun  1  2018 missing
-rw-rw-r-- 1 vagrant vagrant  21106 Jun  1  2018 NEWS
drwxrwxr-x 2 vagrant vagrant   4096 Jun  1  2018 po
-rw-rw-r-- 1 vagrant vagrant   8141 Feb 27  2018 README
drwxrwxr-x 2 vagrant vagrant   4096 Jun  1  2018 sounds
drwxrwxr-x 3 vagrant vagrant   4096 Jun  1  2018 src
-rw-rw-r-- 1 vagrant vagrant    883 Feb 25  2018 TODO
drwxrwxr-x 2 vagrant vagrant   4096 Jun  1  2018 ui
~~~

Antes que nada vamos a crear la carpeta donde queremos que se instale <code>balsa</code>, ya que podríamos instalarlo en <code>/usr/local</code> pero lo vamos a crear en una carpeta a parte para que después tengamos una desinstalación limpia:

~~~
sudo mkdir /opt/balsa
~~~

Y también actualizaremos el sistema:

~~~
sudo apt update && apt upgrade
~~~

Si leemos el documento <code>README</code> e <code>INSTALL</code> nos dirá algunos paquetes que tenemos que instalar y los pasos que tenemos que seguir. El primer paso a seguir es hacer un <code>./configure</code> para que se genere el fichero <code>makefile</code> para poder instalar el programa. Nosotros no vamos a instalar de momento los paquetes requeridos, así que haremos lo siguiente:

~~~
sudo ./configure --prefix=/opt/balsa
~~~

No nos hemos dado cuenta de que no hemos instalado el paquete <code>gcc</code> y <code>make</code>, los cuales son necesarios para poder hacer la compilación:

~~~
sudo apt install gcc make
~~~

Ahora sí, volvemos a ejecutar:

~~~
sudo ./configure --prefix=/opt/balsa
~~~

Leyendo las últimas líneas de error que nos da, vemos lo siguiente:

~~~
...
checking for intltool-update... no
checking for intltool-merge... no
checking for intltool-extract... no
configure: error: The intltool scripts were not found. Please install intltool.
~~~

Con lo cuál tendremos que instalar el paquete:

~~~
sudo apt install intltool
~~~

Volvemos a ejecutar:

~~~
sudo ./configure --prefix=/opt/balsa
~~~

Y las líneas de error nos dan la siguiente información:

~~~
...
configure: error: The pkg-config script could not be found or is too old.
...
~~~

Con lo cual:

~~~
sudo apt install pkg-config
~~~

El siguiente error que nos saldrá al ejecutar de nuevo <code>./configure --prefix=/opt/balsa</code> será el siguiente:

~~~
...
No package 'glib-2.0' found
No package 'gtk+-3.0' found
No package 'gmime-2.6' found
No package 'gio-2.0' found
No package 'gthread-2.0' found
No package 'gnutls' found
...
~~~

Instalamos los siguientes que son los que encuentra nuestra máquina:

~~~
sudo apt install glib-2.0 gtk+-3.0 gmime-2.6 gio-2.0
~~~

El paquete que nos dice de <code>gnutls</code> no lo encontramos, así que vamos a ver en el log si nos da alguna información más precisa:

~~~
sudo less config.log
~~~

Si filtramos por <code>gnutls</code> encontramos la siguiente línea:

~~~
...
Package gnutls was not found in the pkg-config search path
...
~~~

Hemos buscado el error y hemos encontrado que es una librería que tenemos que instalar, para ello:

~~~
sudo apt install libgnutls28-dev
~~~

Si volvemos a ejecutar:

~~~
sudo ./configure --prefix=/opt/balsa
~~~

Nos sale el siguiente error:

~~~
...
No package 'webkit2gtk-4.0' found
...
~~~

Para solucionarlo:

~~~
sudo apt install webkit2gtk-4.0
~~~

Volvemos a ejecutar:

~~~
sudo ./configure --prefix=/opt/balsa
~~~

Y nos damos cuenta de que nos sale otro error más:

~~~
...
checking for NOTIFY... no
configure: error: *** You enabled notify but the library is not found.
...
~~~

El cuál no dice nada en concreto, entonces volvemos a buscar en el log y nos encontramos con la siguiente línea:

~~~
...
Package libnotify was not found in the pkg-config search path
...
~~~

Por ello debemos de instalar la librería de ese paquete también:

~~~
sudo apt install libnotify-dev
~~~

Volvemos a ejecutar el <code>.configure</code> y nos topamos con otro error:

~~~
...
No package 'enchant' found
...
~~~

Viniendo a ser todo lo mismo, si buscamos en el log, veremos la siguiente línea:

~~~
...
Package enchant was not found in the pkg-config search path
...
~~~

Con lo cual esa librería tendremos que instalarla también:

~~~
sudo apt install libenchant-dev
~~~

Al fin, si ejecutamos otra vez el <code>.configure</code>, obtendremos el siguiente resultado:

~~~
================ Final configuration ===================
    Installing into prefix: /opt/balsa
   Enable compile warnings: yes
               HTML widget: webkit2
                 Use GNOME: yes
              Use Canberra: no
                 Use GPGME: no
                  Use LDAP: no
                   Use GSS: no
                Use SQLite: no
             Spell checker: internal
             Use Libnotify:  >= 0.7
         Use GtkSourceView: no
              Use Compface: no
             Use libsecret: no
                   Use gcr: no
~~~

Ahora si listamos la carpeta donde nos encontramos:

~~~
root@compi:/home/vagrant/balsa-2.5.6# ls -l
total 2328
-rw-rw-r-- 1 vagrant vagrant    609 Feb 25  2018 acinclude.m4
-rw-rw-r-- 1 vagrant vagrant  75611 Jun  1  2018 aclocal.m4
-rw-rw-r-- 1 vagrant vagrant   3659 Feb 25  2018 AUTHORS
-rw-r--r-- 1 root    root      6402 Nov  1 14:45 balsa.1
-rw-rw-r-- 1 vagrant vagrant   6410 Feb 25  2018 balsa.1.in
-rw-rw-r-- 1 vagrant vagrant   7759 Jun  1  2018 balsa.appdata.xml
-rw-rw-r-- 1 vagrant vagrant    972 Feb 25  2018 balsa.appdata.xml.in
-rw-r--r-- 1 root    root       273 Nov  1 14:45 balsa.desktop.in
-rw-rw-r-- 1 vagrant vagrant    283 Feb 25  2018 balsa.desktop.in.in
-rw-rw-r-- 1 vagrant vagrant   3533 Feb 25  2018 balsa-mail.lang
-rw-rw-r-- 1 vagrant vagrant   1511 Feb 25  2018 balsa-mail-style.xml
-rw-r--r-- 1 root    root       297 Nov  1 14:45 balsa-mailto-handler.desktop.in
-rw-rw-r-- 1 vagrant vagrant    307 Feb 25  2018 balsa-mailto-handler.desktop.in.in
-rw-r--r-- 1 root    root      4674 Nov  1 14:45 balsa.spec
-rw-rw-r-- 1 vagrant vagrant   4686 Feb 25  2018 balsa.spec.in
-rwxrwxr-x 1 vagrant vagrant    580 Feb 25  2018 bootstrap.sh
-rw-rw-r-- 1 vagrant vagrant 350332 Jun  1  2018 ChangeLog
-rwxr-xr-x 1 vagrant vagrant   7333 Jun  1  2018 compile
-rwxr-xr-x 1 vagrant vagrant  43499 Oct 24  2017 config.guess
-rw-r--r-- 1 root    root      6334 Nov  1 14:45 config.h
-rw-rw-r-- 1 vagrant vagrant   5900 Jun  1  2018 config.h.in
-rw-r--r-- 1 root    root     59368 Nov  1 14:45 config.log
-rwxr-xr-x 1 root    root     77042 Nov  1 14:45 config.status
-rwxr-xr-x 1 vagrant vagrant  36144 Oct 24  2017 config.sub
-rwxrwxr-x 1 vagrant vagrant 654880 Jun  1  2018 configure
-rw-rw-r-- 1 vagrant vagrant  28021 Jun  1  2018 configure.ac
-rw-rw-r-- 1 vagrant vagrant  35147 Jun  1  2018 COPYING
-rwxr-xr-x 1 vagrant vagrant  23566 Jun  1  2018 depcomp
drwxrwxr-x 9 vagrant vagrant   4096 Nov  1 14:45 doc
drwxrwxr-x 2 vagrant vagrant   4096 Jun  1  2018 docs
-rw-rw-r-- 1 vagrant vagrant   3222 Feb 25  2018 gnome-balsa2.png
-rw-rw-r-- 1 vagrant vagrant   3420 Feb 25  2018 HACKING
drwxrwxr-x 5 vagrant vagrant   4096 Nov  1 14:45 images
-rw-rw-r-- 1 vagrant vagrant   2705 Feb 25  2018 INSTALL
-rwxr-xr-x 1 vagrant vagrant  15155 Oct 24  2017 install-sh
-rw-rw-r-- 1 vagrant vagrant      0 Jun  1  2018 intltool-extract.in
-rw-rw-r-- 1 vagrant vagrant      0 Jun  1  2018 intltool-merge.in
-rw-rw-r-- 1 vagrant vagrant      0 Jun  1  2018 intltool-update.in
drwxrwxr-x 4 vagrant vagrant   4096 Nov  1 14:45 libbalsa
drwxrwxr-x 3 vagrant vagrant   4096 Nov  1 14:45 libinit_balsa
drwxrwxr-x 4 vagrant vagrant   4096 Nov  1 14:45 libnetclient
-rwxr-xr-x 1 root    root    339585 Nov  1 14:45 libtool
-rw-r--r-- 1 vagrant vagrant 324404 Feb  7  2016 ltmain.sh
drwxrwxr-x 2 vagrant vagrant   4096 Jun  1  2018 m4
-rw-r--r-- 1 root    root     43976 Nov  1 14:45 Makefile
-rw-rw-r-- 1 vagrant vagrant   2689 Feb 27  2018 Makefile.am
-rw-rw-r-- 1 vagrant vagrant  38281 Jun  1  2018 Makefile.in
-rwxr-xr-x 1 vagrant vagrant   6872 Jun  1  2018 missing
-rw-rw-r-- 1 vagrant vagrant  21106 Jun  1  2018 NEWS
drwxrwxr-x 2 vagrant vagrant   4096 Nov  1 14:45 po
-rw-rw-r-- 1 vagrant vagrant   8141 Feb 27  2018 README
drwxrwxr-x 2 vagrant vagrant   4096 Nov  1 14:45 sounds
drwxrwxr-x 4 vagrant vagrant   4096 Nov  1 14:45 src
-rw-r--r-- 1 root    root        23 Nov  1 14:45 stamp-h1
-rw-rw-r-- 1 vagrant vagrant    883 Feb 25  2018 TODO
drwxrwxr-x 2 vagrant vagrant   4096 Nov  1 14:45 ui
~~~

Vemos que se ha generado el fichero <code>makefile</code>. Ahora vamos a proceder a la compilación y el "linkado"con el comando <code>make</code>:

~~~
sudo make
~~~

Si listamos el directorio <code>/src</code> veremos todos los archivos nuevos que se han generado. Procedemos entonces con la instalación en el directorio que especificamos anteriormente:

~~~
sudo make install
~~~

Y el programa se habrá instalado correctamente:

~~~
root@compi:/home/vagrant/balsa-2.5.6# ls /opt/balsa/
bin  etc  share
~~~

Si ejecutamos <code>balsa</code> desde la terminal no lo encontrará, y eso es porque lo hemos instalado en otra ruta que no tenemos en nuestro <code>$PATH</code>:

~~~
echo $PATH

/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
~~~

Para que lo reconozca cuando lo queramos ejecutar, vamos a crear un enlace simbólico en <code>/usr/local/bin</code> que apunte a nuestro programa:

~~~
sudo ln -s /opt/balsa/bin/* /usr/local/bin/
~~~

Haciéndolo de esta forma podremos hacer una desinstalación mucho más limpia, ya que únicamente borraremos la carpeta <code>/opt/balsa</code> y desinstalaremos las dependencias que hemos instalado para poder generar el <code>makefile</code>.

## Comprobación

Si lo hacemos en una máquina con entorno gráfico, bastaría con ejecutar el comando de <code>balsa</code>:

![4](/assets/img/posts/balsa/4.png)