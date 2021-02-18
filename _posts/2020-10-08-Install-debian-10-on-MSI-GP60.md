---
layout: post
title: Install debian 10 on MSI GP60-2PE
tags: [debian, install, MSI, GP60, 2PE]
---
## Introducción

En este repositorio vamos a explicar como instalar Debian Buster sobre LVM en un MSI GP60.

## Descargas

Para hacer la instalación, tendremos que descargar Debian Buster, podemos encontrarlo [aquí](https://www.debian.org/distrib/). 

## Preparación de PenDrive

Nosotros hemos descargado una imagen ISO y con la ayuda del comando _dd_ en Debian hemos booteado nuestro pendrive de la siguiente forma:

1. Comprobamos el nombre del dispositivo de bloques:
	
	~~~
	lsblk	
	~~~

2. Desmontamos y formateamos ese dispositivo de bloque, en nuestro caso es _/dev/sdc_:

	~~~
	# umount /dev/sdc
	# mkfs.vfat /dev/sdc
	~~~

3. Por último, usamos el comando _dd_, poniendo en _ruta-ISO_ la ruta absoluta del directorio donde tengamos la ISO:
	
	~~~
	# dd if=/ruta-ISO of=/dev/sdc
	~~~

## Instalación

Toca proceder con la instalación en nuestro MSI GP60. Vamos a hacerlo en modo _UEFI_, así que lo primero que tenemos
 que hacer es entrar en la BIOS y asegurarnos que el modo **_UEFI_ está activo** y el **_SecureBoot_ desactivado**, en nuestro
 ordenador entraremos en la BIOS pulsando _Supr_ al arrancar el sistema.

Suponiendo que todos sabemos llegar hasta la _Detección de discos_ en la instalación de Debian, vamos a ir directamente
al particionado. Vamos a centrarnos en el _/dev/sdb_.

![img16](/assets/img/posts/install/img16.png)

Antes que nada, comentar que estos discos no son los originales de mi pc, ya que no podría haber capturado la pantalla en ese instante,
pero lo vamos a hacer de la manera más parecida posible haciendo las capturas en una máquina virtual.

En la imagen anterior podemos observar que tenemos dos discos, cada uno de 8.6GB, en nuestra máquina original tenemos 2 de 500GB, uno HDD y otro SSD,
la instalación la hacemos sobre el espacio libre que tenemos en el _sdb_.

Lo primero, vamos a crear 2 particiones físicas para _/boot_ y _/boot/efi_, ya que pueden dar fallos al instalarla sobre LVM.

![img15](/assets/img/posts/install/img15.png)

Le asignamos el tamaño deseado:

![img14](/assets/img/posts/install/img14.png)

Aunque en este tutorial hemos usado diferentes discos y tamaños, los tamaños recomendados para cada partición son los siguientes:

|Partición||Tamaño|
|--|--|
|/boot||250MB|
|/boot/efi||200MB|
|/||30GB|
|/home||Al gusto|

La cantidad de memoria SWAP a elegir lo dejo a elección de cada uno, ya que depende del Hardware de tu equipo y del trabajo que vayas a hacer.

Seguimos, ponemos partición Primaria:

![img13](/assets/img/posts/install/img13.png)

Y le indicamos en que parte queremos que esté, en principio es indiferente porque luego vamos a trabajar con LVM:

![img12](/assets/img/posts/install/img12.png)

Le damos un formato de _ext4_ y le decimos que será la partición de _/boot_:

![img11](/assets/img/posts/install/img11.png)

Hacemos lo mismo para la partición _/boot/efi_ asignandole a esta _fat32_ como formato y entramos en el
_Gestor de Volúmenes Lógicos (LVM)_:

![img10](/assets/img/posts/install/img10.png)

Creamos un nuevo grupo de volúmenes:

![img9](/assets/img/posts/install/img9.png)

Le asignamos el nombre deseado:

![img8](/assets/img/posts/install/img8.png)

Los dispositivos que queremos en ese volumen:

![img3](/assets/img/posts/install/img3.png)

Y confirmamos guardar los cambios en disco:

![img7](/assets/img/posts/install/img7.png)

Ahora creamos un volumen lógico:

![img6](/assets/img/posts/install/img6.png)

Le decimos que lo queremos de nuestro grupo de volúmenes:

![img5](/assets/img/posts/install/img5.png)

Le damos un nombre significativo, ahora lo haremos para la partición _Raiz_:

![img4](/assets/img/posts/install/img4.png)

Le damos el tamaño a nuestro volumen lógico (acordarse de los tamaños recomendados):

![img2](/assets/img/posts/install/img2.png)

Y hacemos lo mismo para la partición _/home_. Una vez que lo hacemos, debemos de darle formato y decirle donde irán
montadas. Finalizamos el particionado y escribimos los cambios en el disco:

![img1](/assets/img/posts/install/img1.png)

Con esto lo único que tendríamos que esperar sería que se completara la instalación y tendríamos nuestro
Debian 10 instalado.

## Post-Instalación

Una vez que ya tenemos Debian instalado, no nos funcionará la Wifi, ya que por defecto
no viene instalada en el pc.

Para comprobar cuál es el firmware que nos hará falta, basta con poner lo siguiente:

![imgwifi](/assets/img/posts/install/imgwifi.png)

Y como vemos, nos dice que el firmware que nos hace falta:

~~~
iwlwifi-3160-17 is required
~~~

Para la descarga del paquete, nos vamos a ir a la 
[página oficial de paquetes de Debian](https://packages.debian.org/buster/all/firmware-iwlwifi/download)
y descargaremos el paquete del sitio _ftp.es.debian.org/debian_.

Una vez que lo tengamos, ejecutamos:

~~~
# dpkg -i firmware-iwlwifi_20190114-2_all.deb
~~~

Reiniciamos nuestra máquina y ya tendremos wifi en nuestro equipo. Y podremos comprobar las particiones ejecutando 
lo siguiente:

~~~
sdb                                                                                     
├─sdb1          ext4        BOOT  e09d56a3-e45b-47cb-b727-00682b9f3171    329,7M    21% /boot
├─sdb2          LVM2_member       g3Q490-n0cK-UHit-98nP-6MO0-YJy0-9suhgs                
│ ├─debian-raiz ext4              bd18d413-c1ba-4b68-95d3-ec9453cc34eb     20,1G    22% /
│ └─debian-home ext4              25a67dcf-270b-4bf7-9fc4-c6af7260902d     98,9G     5% /home
└─sdb3          vfat              B716-D9EC                                 471M     1% /boot/efi
~~~