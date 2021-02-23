---
layout: post
title: Instalación y manejo de BTRFS
tags: [all, BTRFS, sistema de ficheros, RAID]
---
# Introducción

Buenas, en este post vamos a instalar y configurar el software de BTRFS, un sistema de ficheros avanzados para ver su uso y funcionamiento.

## Configuración escenario

Hemos creado una máquina en OpenStack y le hemos añadido 4 volúmenes de prueba de diferentes tamaños cada uno:

~~~
lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    254:0    0  20G  0 disk 
└─vda1 254:1    0  20G  0 part /
vdb    254:16   0   1G  0 disk /mnt
vdd    254:48   0   2G  0 disk 
vde    254:64   0   3G  0 disk 
vdf    254:80   0   4G  0 disk
~~~

## Instalación

Vamos a instalar `Btrfs`, para ello, actualizamos la máquina e instalamos:

~~~
sudo apt update && sudo apt upgrade -y
sudo apt install btrfs-tools -y
~~~

## Crear sistema Btrfs

Para dar formato a uno de los discos con el nuevo sistema de ficheros:

~~~
sudo mkfs.btrfs /dev/vdb
btrfs-progs v4.20.1 
See http://btrfs.wiki.kernel.org for more information.

Label:              (null)
UUID:               9a11dd38-95e9-46c7-892e-180eab7f211f
Node size:          16384
Sector size:        4096
Filesystem size:    1.00GiB
Block group profiles:
  Data:             single            8.00MiB
  Metadata:         DUP              51.19MiB
  System:           DUP               8.00MiB
SSD detected:       no
Incompat features:  extref, skinny-metadata
Number of devices:  1
Devices:
   ID        SIZE  PATH
    1     1.00GiB  /dev/vdb
~~~

Y podemos comprobarlo:

~~~
lsblk -f
NAME   FSTYPE LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINT
vda                                                                     
└─vda1 ext4         9659e5d4-dd87-42af-bf70-0bb6f7b2e31b   17.8G     5% /
vdb    btrfs        9a11dd38-95e9-46c7-892e-180eab7f211f                
vdd                                                                     
vde                                                                     
vdf
~~~

## Gestion discos

Vamos a dar formato a dos discos para ver que ocurre, para ello:

~~~
sudo mkfs.btrfs -f /dev/vdb /dev/vdd
btrfs-progs v4.20.1 
See http://btrfs.wiki.kernel.org for more information.

Label:              (null)
UUID:               c4ed17de-6443-4750-b59b-b9db8aa9d589
Node size:          16384
Sector size:        4096
Filesystem size:    3.00GiB
Block group profiles:
  Data:             RAID0           307.12MiB
  Metadata:         RAID1           153.56MiB
  System:           RAID1             8.00MiB
SSD detected:       no
Incompat features:  extref, skinny-metadata
Number of devices:  2
Devices:
   ID        SIZE  PATH
    1     1.00GiB  /dev/vdb
    2     2.00GiB  /dev/vdd
~~~

Hemos usado la opción `-f` porque uno de los discos ya le dimos formato anteriormente. Si nos damos cuenta, lo que hemos conseguido hacer es unir los dos dispositivos como si fueran uno sólo, en lo que parece un RAID híbrido entre RAID 1 y RAID 0. Podemos comprobar que los dos dispositivos tienen el mismo UID de la siguiente forma:

~~~
lsblk -f
NAME   FSTYPE LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINT
vda                                                                     
└─vda1 ext4         9659e5d4-dd87-42af-bf70-0bb6f7b2e31b   17.8G     5% /
vdb    btrfs        c4ed17de-6443-4750-b59b-b9db8aa9d589                
vdd    btrfs        c4ed17de-6443-4750-b59b-b9db8aa9d589                
vde                                                                     
vdf
~~~

Podemos montar cualquiera de los dos dispositivos para verlo:

~~~
sudo mount /dev/vdb /mnt/
~~~

Veremos que la capacidad es la de los dos discos juntos:

~~~
df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            235M     0  235M   0% /dev
tmpfs            49M  1.6M   47M   4% /run
/dev/vda1        20G  1.1G   18G   6% /
tmpfs           243M     0  243M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           243M     0  243M   0% /sys/fs/cgroup
tmpfs            49M     0   49M   0% /run/user/1000
/dev/vdb        3.0G   17M  1.7G   1% /mnt
~~~

## RAID 1

Vamos a configurar un RAID 1 con sistema de tolerancia a fallos, para ello:

~~~
sudo mkfs.btrfs -f -d raid1 -m raid1 /dev/vdf /dev/vdd /dev/vde
btrfs-progs v4.20.1 
See http://btrfs.wiki.kernel.org for more information.

Label:              (null)
UUID:               aef23e40-53f8-43a5-b849-16ecb3d938f2
Node size:          16384
Sector size:        4096
Filesystem size:    9.00GiB
Block group profiles:
  Data:             RAID1           460.75MiB
  Metadata:         RAID1           460.75MiB
  System:           RAID1             8.00MiB
SSD detected:       no
Incompat features:  extref, skinny-metadata
Number of devices:  3
Devices:
   ID        SIZE  PATH
    1     4.00GiB  /dev/vdf
    2     2.00GiB  /dev/vdd
    3     3.00GiB  /dev/vde
~~~

Como podemos apreciar se ha creado un RAID1 correctamente, es más, hemos podido aprovechar todo el espacio de los discos aunque sean de diferente tamaño. En el caso de que hubieramos hecho un RAID1 con el software `mdadm` todos los dispositivos de bloques deberían de tener el mismo tamaño, o en su defecto, se escogería la capacidad del disco menor.

Podemos mostrar los dispositivos de la siguiente forma:

~~~
sudo btrfs filesystem show

Label: none  uuid: aef23e40-53f8-43a5-b849-16ecb3d938f2
	Total devices 3 FS bytes used 128.00KiB
	devid    1 size 4.00GiB used 921.50MiB path /dev/vdf
	devid    2 size 2.00GiB used 468.75MiB path /dev/vdd
	devid    3 size 3.00GiB used 468.75MiB path /dev/vde
~~~

### Añadir disco a RAID

Para añadir un disco al RAID, podemos hacerlo de la siguiente forma, primero tenemos que montar el RAID:

~~~
sudo mount /dev/vdf /mnt/
~~~

Y después añadimos el disco al RAID:

~~~
sudo btrfs device add -f /dev/vdb /mnt/
~~~

Mostramos otra vez los discos:

~~~
sudo btrfs filesystem show
Label: none  uuid: aef23e40-53f8-43a5-b849-16ecb3d938f2
	Total devices 4 FS bytes used 256.00KiB
	devid    1 size 4.00GiB used 921.50MiB path /dev/vdf
	devid    2 size 2.00GiB used 468.75MiB path /dev/vdd
	devid    3 size 3.00GiB used 468.75MiB path /dev/vde
	devid    4 size 1.00GiB used 0.00B path /dev/vdb
~~~

Se ha añadido correctamente pero no está haciendo uso de su espacio, para ello tenemos que activar el balanceo de carga para que se reparta la información entre todos los discos:

~~~
sudo btrfs balance start --full-balance /mnt/
~~~

Ahora vamos a escribir en el disco para comprobar que se está haciendo correctamente el balanceo y cuanto espacio puede soportar nuestro RAID:

~~~
sudo dd if=/dev/zero of=/mnt/prueba
dd: writing to '/mnt/prueba': No space left on device
9890530+0 records in
9890529+0 records out
5063950848 bytes (5.1 GB, 4.7 GiB) copied, 177.7 s, 28.5 MB/s
~~~

Y si comprobamos los dispositivos:

~~~
sudo btrfs filesystem show
Label: none  uuid: aef23e40-53f8-43a5-b849-16ecb3d938f2
	Total devices 4 FS bytes used 4.72GiB
	devid    1 size 4.00GiB used 4.00GiB path /dev/vdf
	devid    2 size 2.00GiB used 2.00GiB path /dev/vdd
	devid    3 size 3.00GiB used 3.00GiB path /dev/vde
	devid    4 size 1.00GiB used 1023.00MiB path /dev/vdb
~~~

Podemos observar que ha rellenado todos los discos con su capacidad máxima. Si hubiéramos hecho el RAID con mdadm sólo podríamos haber escrito 1GB, ya que es la capacidad máxima soportada por el menos disco.

### Comando para saber integridad de datos en raid

Para hacer un chequeo de la integridad de los datos del raid, podemos ejecutar lo siguiente:

~~~
sudo btrfs scrub start /mnt/
~~~

Y es un proceso que se ejecuta en segundo plano el cual podemos ver su estado de la siguiente forma:

~~~
sudo btrfs scrub status /mnt/
scrub status for aef23e40-53f8-43a5-b849-16ecb3d938f2
	scrub started at Thu Jan 21 17:18:38 2021, running for 00:00:20
	total bytes scrubbed: 2.08GiB with 0 errors
~~~

Cuando acabe nos saldrá el siguiente mensaje:

~~~
sudo btrfs scrub status /mnt/
scrub status for aef23e40-53f8-43a5-b849-16ecb3d938f2
	scrub started at Fri Jan 22 07:37:06 2021 and finished after 00:01:34
	total bytes scrubbed: 9.44GiB with 0 errors
~~~

El cual nos ha dicho que no ha habido ningún error.

### Fallo en algún disco

Vamos a quitar un disco haciéndo creer que ha fallado y ejecutamos la instrucción para ver el raid:

~~~
sudo btrfs filesystem show
Label: none  uuid: aef23e40-53f8-43a5-b849-16ecb3d938f2
	Total devices 4 FS bytes used 4.72GiB
	devid    1 size 4.00GiB used 4.00GiB path /dev/vdf
	devid    2 size 2.00GiB used 2.00GiB path /dev/vdd
	devid    3 size 3.00GiB used 3.00GiB path /dev/vde
	*** Some devices missing
~~~

Como vemos ejecutando el comando anterior tenemos errores porque un disco no funciona:

~~~
sudo btrfs scrub status /mnt/
scrub status for aef23e40-53f8-43a5-b849-16ecb3d938f2
	scrub started at Fri Jan 22 08:27:21 2021 and finished after 00:03:08
	total bytes scrubbed: 9.44GiB with 261874 errors
	error details: read=261872 super=2
	corrected errors: 0, uncorrectable errors: 261872, unverified errors: 0
~~~

### Sustitución de disco dañado

Para poder sustituir el disco hemos añadido uno nuevo:

~~~
lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    254:0    0  20G  0 disk 
└─vda1 254:1    0  20G  0 part /
vdc    254:32   0   1G  0 disk /mnt
vdd    254:48   0   2G  0 disk 
vde    254:64   0   3G  0 disk 
vdf    254:80   0   4G  0 disk
~~~

Y vamos a ejecutar la siguiente orden:

~~~
sudo btrfs replace start 4 /dev/vdc /mnt/
~~~

Seguidamente podremos ver su progreso como muestro a continuación:

~~~
sudo btrfs replace status /mnt/
0.0% done, 0 write errs, 0 uncorr. read errs
~~~

Y cuando finalize nos saldrá el siguiente mensaje:

~~~
sudo btrfs replace status /mnt/
Started on 22.Jan 09:08:01, finished on 22.Jan 09:11:38, 0 write errs, 0 uncorr. read errs
~~~

### Comprobación

Vemos que todo está correcto:

~~~
sudo btrfs filesystem show
Label: none  uuid: aef23e40-53f8-43a5-b849-16ecb3d938f2
	Total devices 4 FS bytes used 4.72GiB
	devid    1 size 4.00GiB used 4.00GiB path /dev/vdf
	devid    2 size 2.00GiB used 2.00GiB path /dev/vdd
	devid    3 size 3.00GiB used 3.00GiB path /dev/vde
	devid    4 size 1.00GiB used 1023.00MiB path /dev/vdc
~~~

Para comprobar el estado del RAID:

~~~
sudo btrfs scrub start /mnt/
scrub started on /mnt/, fsid aef23e40-53f8-43a5-b849-16ecb3d938f2 (pid=9150)
~~~

Y al finalizar:

~~~
sudo btrfs scrub status /mnt/
scrub status for aef23e40-53f8-43a5-b849-16ecb3d938f2
	scrub started at Fri Jan 22 09:52:45 2021 and finished after 00:01:32
	total bytes scrubbed: 9.44GiB with 0 errors
~~~

## Compresión

Existen dos formas de compresión al vuelo en BTRFS, ZLIB y LZO. Este último es más rápido pero comprime menos. Nosotros usaremos ZLIB, que comprime con mayor detalle pero más lento.

Para poder ejecutar la comprensión al vuelo vamos a formatear uno de los discos de 4G que tenemos:

~~~
sudo mkfs.btrfs -f /dev/vde 
btrfs-progs v4.20.1 
See http://btrfs.wiki.kernel.org for more information.

Label:              (null)
UUID:               0cbafa86-b631-423e-9050-1119842d3b01
Node size:          16384
Sector size:        4096
Filesystem size:    4.00GiB
Block group profiles:
  Data:             single            8.00MiB
  Metadata:         DUP             204.75MiB
  System:           DUP               8.00MiB
SSD detected:       no
Incompat features:  extref, skinny-metadata
Number of devices:  1
Devices:
   ID        SIZE  PATH
    1     4.00GiB  /dev/vde
~~~

Y lo vamos a montar de la siguiente forma:

~~~
$ sudo mount -o compress=zlib /dev/vde /mnt/

$ lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    254:0    0  20G  0 disk 
└─vda1 254:1    0  20G  0 part /
vdb    254:16   0   1G  0 disk 
vdc    254:32   0   2G  0 disk 
vdd    254:48   0   3G  0 disk 
vde    254:64   0   4G  0 disk /mnt
~~~

Ahora vamos a rellenar el disco con gran cantidad de datos:

~~~
 sudo dd if=/dev/zero of=/mnt/prueba
^C175804802+0 records in
175804802+0 records out
90012058624 bytes (90 GB, 84 GiB) copied, 1281.03 s, 70.3 MB/s
~~~

Y podemos comprobar que nuestro disco es de tan sólo 4GB:

~~~
sudo btrfs filesystem show /mnt/
Label: none  uuid: 0cbafa86-b631-423e-9050-1119842d3b01
	Total devices 1 FS bytes used 2.78GiB
	devid    1 size 4.00GiB used 3.27GiB path /dev/vde
~~~

Pero que los datos que ahora mismo tiene en él son los siguientes:

~~~
du -h /mnt/
84G	/mnt/
~~~

84GB es lo que tiene comprimidos de tal forma que sólo ocupan 3.27GiB, y cada vez que queramos acceder a ese dato automáticamente BTRFS lo descomprimirá y lo volverá a comprimir.

## Subvolumenes y snapshots

Vamos a crear subvolumenes, los cuales podemos montar en varios directorios para poder usarlos simúltaneamente, por ejemplo, tener montado uno en /home y otro en /etc y gracias a los snapshots tener una copia de cada uno de los directorios, para ello lo primero que haremos será crear el grupo de volúmenes:

~~~
sudo mkfs.btrfs /dev/vdc /dev/vdd /dev/vdb -f -L 'sub'
btrfs-progs v4.20.1 
See http://btrfs.wiki.kernel.org for more information.

Label:              sub
UUID:               a4328666-4496-446e-86ad-a723db37aad6
Node size:          16384
Sector size:        4096
Filesystem size:    6.00GiB
Block group profiles:
  Data:             RAID0           614.25MiB
  Metadata:         RAID1           307.19MiB
  System:           RAID1             8.00MiB
SSD detected:       no
Incompat features:  extref, skinny-metadata
Number of devices:  3
Devices:
   ID        SIZE  PATH
    1     2.00GiB  /dev/vdc
    2     3.00GiB  /dev/vdd
    3     1.00GiB  /dev/vdb
~~~

Montamos el grupo de volúmenes:

~~~
sudo mount /dev/vdc /mnt/
~~~

Comprobamos:

~~~
sudo btrfs filesystem show
Label: 'sub'  uuid: a4328666-4496-446e-86ad-a723db37aad6
	Total devices 3 FS bytes used 128.00KiB
	devid    1 size 2.00GiB used 511.94MiB path /dev/vdc
	devid    2 size 3.00GiB used 212.75MiB path /dev/vdd
	devid    3 size 1.00GiB used 519.94MiB path /dev/vdb
~~~

Y ahora creamos varios subvolúmenes:

~~~
sudo btrfs subvolume create /mnt/sub1
sudo btrfs subvolume create /mnt/sub2
sudo btrfs subvolume create /mnt/sub3
~~~

Los listamos para comprobar que se han creado correctamente:

~~~
sudo btrfs subvolume list /mnt/
ID 258 gen 7 top level 5 path sub1
ID 259 gen 8 top level 5 path sub2
ID 260 gen 9 top level 5 path sub3
~~~

Para poder montar dichos subvolúmenes tenemos que saber su ID como hemos visto anteriormente, lo vamos a montar en /media:

~~~
sudo mount -o subvolid=258 /dev/vdc /media
~~~

Creamos un fichero en /media:

~~~
sudo touch /media/fichero.txt
~~~

Y si comprobamos en /mnt/sub1:

~~~
ls /mnt/sub1/
fichero.txt
~~~

Podemos también comprobar qué volumen está montado en el directorio:

~~~
sudo btrfs subvolume show /media
sub1
	Name: 			sub1
	UUID: 			2861b2ba-3fa4-f745-9b05-6d4e135f7d04
	Parent UUID: 		-
	Received UUID: 		-
	Creation time: 		2021-01-22 16:23:18 +0000
	Subvolume ID: 		258
	Generation: 		11
	Gen at creation: 	7
	Parent ID: 		5
	Top level ID: 		5
	Flags: 			-
	Snapshot(s):
~~~

### Snapshots

Para poder hacerle un snapshots a un subvolumen:

~~~
sudo btrfs subvolume snapshot /mnt/sub1 /mnt/sub1.snap
Create a snapshot of '/mnt/sub1' in '/mnt/sub1.snap'
~~~

Y para poder restaurar dicha copia:

~~~
sudo mv /mnt/sv1.snap /mnt/sv1
~~~

## Redimensionado

Podemos redimensionar el cualquier momento un sistema de btrfs, suponiendo que tenemos lo siguiente:

~~~
sudo btrfs filesystem show /mnt/
Label: 'red'  uuid: 6abfd83c-3dcd-4fd5-b5d9-e5766e72c2cf
	Total devices 1 FS bytes used 128.00KiB
	devid    1 size 2.00GiB used 228.75MiB path /dev/vdc
~~~

Vamos a redimensionar a 1G, para ello:

~~~
sudo btrfs filesystem resize -1g /mnt/
Resize '/mnt/' of '-1g'
~~~

Y comprobamos:

~~~
sudo btrfs filesystem show /mnt/
Label: 'red'  uuid: 6abfd83c-3dcd-4fd5-b5d9-e5766e72c2cf
	Total devices 1 FS bytes used 192.00KiB
	devid    1 size 1.00GiB used 228.75MiB path /dev/vdc
~~~

Podemos volver a darle su valor máximo de siguiente forma:

~~~
sudo btrfs filesystem resize max /mnt/
Resize '/mnt/' of 'max'
~~~

Y lo volvemos a comprobar:

~~~
sudo btrfs filesystem show /mnt/
Label: 'red'  uuid: 6abfd83c-3dcd-4fd5-b5d9-e5766e72c2cf
	Total devices 1 FS bytes used 192.00KiB
	devid    1 size 2.00GiB used 228.75MiB path /dev/vdc
~~~

## Copy on write

Vamos a hacer Copy on write, lo cuál nos permite copiar varios ficheros iguales pero sólo ocupando el espacio de uno. Tenemos el siguiente escenario:

~~~
sudo btrfs filesystem show /mnt/
Label: 'red'  uuid: 6abfd83c-3dcd-4fd5-b5d9-e5766e72c2cf
	Total devices 1 FS bytes used 192.00KiB
	devid    1 size 2.00GiB used 228.75MiB path /dev/vdc
~~~

Para poder utilizar esta utilidad, vamos a crear un fichero de 1G:

~~~
sudo dd if=/dev/urandom of=/mnt/fichero1 bs=1M count=1000
1000+0 records in
1000+0 records out
1048576000 bytes (1.0 GB, 1000 MiB) copied, 8.50991 s, 123 MB/s
~~~

Si vemos su tamaño:

~~~
ls -lh /mnt/
total 1000M
-rw-r--r-- 1 root root 1000M Jan 22 17:05 fichero1
~~~

Vamos a copiar varios ficheros, nuestro espacio máximo es de 2GB:

~~~
sudo cp --reflink=always fichero1 fichero2
sudo cp --reflink=always fichero1 fichero3
sudo cp --reflink=always fichero1 fichero4
sudo cp --reflink=always fichero1 fichero5
~~~

Nos ha dejado perfectamente, ya que si vemos en realidad cuánto espacio está consumiendo:

~~~
df -h /mnt/
Filesystem      Size  Used Avail Use% Mounted on
/dev/vdc        2.0G 1019M  826M  56% /mnt
~~~