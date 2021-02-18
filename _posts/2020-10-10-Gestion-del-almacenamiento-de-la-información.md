---
layout: post
title: Gestion del almacenamiento de la información
tags: [RAID, almacenamiento, Vagrant]
---
# Introducción

Buenas, en esta práctica vamos a hacer varios ejercicios para crear un RAID 1 en una máquina Debian y gestionarlo. 

Un RAID 1 nos va a permitir tener dos discos con los mismos datos y si uno de ellos falla, el sistema seguirá funcionando correctamente y podremos sustituir el disco dañado.

## RAID 1
### Creacción de máquina Debian con Vagrant

Para crear la máquina vamos a usar Vagrant, una herramienta la cuál podemos aprender un poco en [este post](https://acabe10.github.io/acabe10.github.io/2020/10/10/Introducci%C3%B3n-a-Vagrant.html), para crear usamos el siguiente _VagrantFile_:

~~~
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
 disco1 = '.vagrant/disco1.vdi'
 disco2 = '.vagrant/disco2.vdi'
	  config.vm.define :nodo1 do |nodo1|
	    nodo1.vm.box = "debian/buster64" 
	    nodo1.vm.hostname = "nodo1" 
	    nodo1.vm.network :public_network,:bridge=>"wlp5s0" 
	    nodo1.vm.network :private_network, ip: "10.1.1.10" 
	    nodo1.vm.provider :virtualbox do |v|
	        if not File.exist?(disco1)
	                v.customize ["createhd", "--filename", disco1, "--size", 1024]
	                v.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", 1, "--device", 0, "--type", "hdd", "--medium", disco1]
	        end
	        if not File.exist?(disco2)
	                v.customize ["createhd", "--filename", disco2, "--size", 1024]
	                v.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", 2, "--device", 0, "--type", "hdd", "--medium", disco2]
	        end
	    nodo1.vm.provision "shell",
	      run: "always",
	      inline: "apt install mdadm -y" 
	    end
	  end
end
~~~

El anterior _VagrantFile_ nos creará una máquina con dos discos de 1GB y al iniciar la máquina, nos instalará el software _mdadm_ que es el encargado de administrar RAIDS.

### Creación de RAID 1

Para crear el RAID ejecutamos:

~~~
# mdadm --create /dev/md1 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
~~~

### Comprobar estado de RAID

Para comprobar los discos:

~~~
root@nodo1:/home/vagrant# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda      8:0    0 19.8G  0 disk  
├─sda1   8:1    0 18.8G  0 part  /
├─sda2   8:2    0    1K  0 part  
└─sda5   8:5    0 1021M  0 part  [SWAP]
sdb      8:16   0    1G  0 disk  
└─md1    9:1    0 1022M  0 raid1 
sdc      8:32   0    1G  0 disk  
└─md1    9:1    0 1022M  0 raid1
~~~

Y para comprobar el estado del RAID:

~~~
root@nodo1:/home/vagrant# mdadm --detail /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Mon Sep 30 10:33:46 2019
        Raid Level : raid1
        Array Size : 1046528 (1022.00 MiB 1071.64 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Mon Sep 30 10:33:51 2019
             State : clean 
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

Consistency Policy : resync

              Name : nodo1:1  (local to host nodo1)
              UUID : fcb60778:1b3ca292:626ee47b:d8445cb9
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
~~~

### Crear vólumen en RAID

Para crear vólumenes, vamos a usar la herramienta _fdisk_:

~~~
root@nodo1:/home/vagrant# fdisk /dev/md1
~~~

Y creamos una partición de 500MB:

~~~
root@nodo1:/home/vagrant# lsblk
NAME      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda         8:0    0 19.8G  0 disk  
├─sda1      8:1    0 18.8G  0 part  /
├─sda2      8:2    0    1K  0 part  
└─sda5      8:5    0 1021M  0 part  [SWAP]
sdb         8:16   0    1G  0 disk  
└─md1       9:1    0 1022M  0 raid1 
  └─md1p1 259:1    0  500M  0 part  
sdc         8:32   0    1G  0 disk  
└─md1       9:1    0 1022M  0 raid1 
  └─md1p1 259:1    0  500M  0 part
~~~

### Dar un formato a la partición

Para ello ejecutamos la siguiente instrucción:

~~~
root@nodo1:/home/vagrant# mkfs.ext3 /dev/md1p1
mke2fs 1.44.5 (15-Dec-2018)
Creating filesystem with 512000 1k blocks and 128016 inodes
Filesystem UUID: bf5ca2b2-42c9-4dbe-b914-b2448620621f
Superblock backups stored on blocks: 
    8193, 24577, 40961, 57345, 73729, 204801, 221185, 401409

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done
~~~

### Montar partición y hacerla permanente

Para poder trabajar con esa partición, tendremos que montarla:

~~~
root@nodo1:/home/vagrant# mkdir /mnt/raid1
root@nodo1:/home/vagrant# mount -t ext3 /dev/md1p1 /mnt/raid1
root@nodo1:/home/vagrant# lsblk
NAME      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda         8:0    0 19.8G  0 disk  
├─sda1      8:1    0 18.8G  0 part  /
├─sda2      8:2    0    1K  0 part  
└─sda5      8:5    0 1021M  0 part  [SWAP]
sdb         8:16   0    1G  0 disk  
└─md1       9:1    0 1022M  0 raid1 
  └─md1p1 259:1    0  500M  0 part  /mnt/raid1
sdc         8:32   0    1G  0 disk  
└─md1       9:1    0 1022M  0 raid1 
  └─md1p1 259:1    0  500M  0 part  /mnt/raid1
~~~

Probamos que podemos crear un fichero:

~~~
root@nodo1:/mnt/raid1# touch prueba_raid1
~~~

Para que sea permanente debemos de editar el fichero _/etc/fstab_ añadiendo la siguiente información:

~~~
UUID=bf5ca2b2-42c9-4dbe-b914-b2448620621f    /mnt/raid1    ext3    defaults 0 2
~~~

Donde UUID lo prodríamos obtener haciendo un simple comando:

~~~
root@nodo1:/home/vagrant# lsblk -f
NAME      FSTYPE            LABEL   UUID                                 FSAVAIL FSUSE% MOUNTPOINT
sda                                                                                     
├─sda1    ext4                      b9ffc3d1-86b2-4a2c-a8be-f2b2f4aa4cb5   16.4G     6% /
├─sda2                                                                                  
└─sda5    swap                      f8f6d279-1b63-4310-a668-cb468c9091d8                [SWAP]
sdb       linux_raid_member nodo1:1 ffb909e3-2fe4-b865-6b53-ff820e7b0aa8                
└─md1                                                                                   
  └─md1p1 ext3                      bf5ca2b2-42c9-4dbe-b914-b2448620621f  448.9M     0% /mnt/raid1
sdc       linux_raid_member nodo1:1 ffb909e3-2fe4-b865-6b53-ff820e7b0aa8                
└─md1                                                                                   
  └─md1p1 ext3                      bf5ca2b2-42c9-4dbe-b914-b2448620621f  448.9M     0% /mnt/raid1
~~~

### Forzar fallo de disco

Comprobamos que todo está funcionando correctamente:

~~~
root@nodo1:/mnt/raid1# mdadm --detail /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Sun Oct 13 10:07:12 2019
        Raid Level : raid1
        Array Size : 1046528 (1022.00 MiB 1071.64 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Sun Oct 13 10:17:54 2019
             State : clean 
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

Consistency Policy : resync

              Name : nodo1:1  (local to host nodo1)
              UUID : ffb909e3:2fe4b865:6b53ff82:0e7b0aa8
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
~~~

Forzamos el fallo de un disco:

~~~
root@nodo1:/mnt/raid1# mdadm /dev/md1 -f /dev/sdb
mdadm: set /dev/sdb faulty in /dev/md1
~~~

Y volvemos a comprobar el estado del raid:

~~~
root@nodo1:/mnt/raid1# mdadm --detail /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Sun Oct 13 10:07:12 2019
        Raid Level : raid1
        Array Size : 1046528 (1022.00 MiB 1071.64 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Sun Oct 13 10:31:49 2019
             State : clean, degraded 
    Active Devices : 1
   Working Devices : 1
    Failed Devices : 1
     Spare Devices : 0

Consistency Policy : resync

              Name : nodo1:1  (local to host nodo1)
              UUID : ffb909e3:2fe4b865:6b53ff82:0e7b0aa8
            Events : 19

    Number   Major   Minor   RaidDevice State
       -       0        0        0      removed
       1       8       32        1      active sync   /dev/sdc

       0       8       16        -      faulty   /dev/sdb
~~~

Como hemos podido observar, nos dice que uno de los elementos del raid está fallando y el otro está activo. Y aún así podemos seguir accediendo al fichero, pero nos dice que esta "degraded", es decir, que no está funcionando correctamente al ser un raid 1 con un sólo disco.

~~~
root@nodo1:/mnt/raid1# cat prueba_raid1 
funcionando
~~~

### Recuperar estado de RAID

Para recuperar el estado del raid, primero debemos eliminar el disco en mal estado:

~~~
root@nodo1:/mnt/raid1# mdadm /dev/md1 -r /dev/sdb
~~~

Volvemos a añadir el disco:

~~~
root@nodo1:/mnt/raid1# mdadm /dev/md1 -a /dev/sdb
~~~

Y comprobamos el estado:

~~~
root@nodo1:/mnt/raid1# mdadm --detail /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Sun Oct 13 10:07:12 2019
        Raid Level : raid1
        Array Size : 1046528 (1022.00 MiB 1071.64 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Sun Oct 13 10:44:05 2019
             State : clean 
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

Consistency Policy : resync

              Name : nodo1:1  (local to host nodo1)
              UUID : ffb909e3:2fe4b865:6b53ff82:0e7b0aa8
            Events : 47

    Number   Major   Minor   RaidDevice State
       2       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
~~~

Y comprobamos si accedemos al fichero:

~~~
root@nodo1:/mnt/raid1# cat prueba_raid1 
funcionando
~~~

### Añadir otro disco al RAID 1

Para ello, hemos editado el fichero _VagrantFile_. Añadimos tercer disco:

~~~
root@nodo1:/home/vagrant# mdadm /dev/md1 -a /dev/sdd 
mdadm: added /dev/sdd
~~~

Y comprobamos:

~~~
root@nodo1:/home/vagrant# lsblk
NAME      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda         8:0    0 19.8G  0 disk  
├─sda1      8:1    0 18.8G  0 part  /
├─sda2      8:2    0    1K  0 part  
└─sda5      8:5    0 1021M  0 part  [SWAP]
sdb         8:16   0    1G  0 disk  
└─md1       9:1    0 1022M  0 raid1 
  └─md1p1 259:1    0  500M  0 part  
sdc         8:32   0    1G  0 disk  
└─md1       9:1    0 1022M  0 raid1 
  └─md1p1 259:1    0  500M  0 part  
sdd         8:48   0    1G  0 disk  
└─md1       9:1    0 1022M  0 raid1 
  └─md1p1 259:1    0  500M  0 part 
~~~

~~~
root@nodo1:/home/vagrant# mdadm --detail /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Sun Oct 13 11:23:44 2019
        Raid Level : raid1
        Array Size : 1046528 (1022.00 MiB 1071.64 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 2
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Sun Oct 13 11:26:00 2019
             State : clean 
    Active Devices : 2
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 1

Consistency Policy : resync

              Name : nodo1:1  (local to host nodo1)
              UUID : 7caaa67c:40e4bfcc:2f1c7bbb:c2d791b6
            Events : 18

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc

       2       8       48        -      spare   /dev/sdd
~~~

Este disco se mantendrá como un disco de repuesto y en caso de que fallen alguno de los otros dos, empezará a funcionar. Podemos sustituir un disco por uno de la ráiz de la siguiente forma:

~~~
	root@nodo1:/home/vagrant# mdadm --manage /dev/md1 --replace /dev/sdb --with /dev/sdd
mdadm: Marked /dev/sdb (device 0 in /dev/md1) for replacement
mdadm: Marked /dev/sdd in /dev/md1 as replacement for device 0
root@nodo1:/home/vagrant# mdadm --detail /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Sun Oct 13 11:23:44 2019
        Raid Level : raid1
        Array Size : 1046528 (1022.00 MiB 1071.64 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 2
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Sun Oct 13 11:53:45 2019
             State : clean, recovering 
    Active Devices : 2
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 1

Consistency Policy : resync

    Rebuild Status : 57% complete

              Name : nodo1:1  (local to host nodo1)
              UUID : 7caaa67c:40e4bfcc:2f1c7bbb:c2d791b6
            Events : 24

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       2       8       48        0      spare rebuilding   /dev/sdd
       1       8       32        1      active sync   /dev/sdc
~~~

O también podemos añadir otro disco al array ejecutando lo siguiente:

~~~
root@nodo1:/home/vagrant# mdadm --grow /dev/md1 --raid-devices=3
~~~

Y comprobamos:

~~~
	root@nodo1:/home/vagrant# mdadm --detail /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Mon Oct 14 19:48:58 2019
        Raid Level : raid1
        Array Size : 1046528 (1022.00 MiB 1071.64 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 3
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Mon Oct 14 19:50:37 2019
             State : clean 
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 0

Consistency Policy : resync

              Name : nodo1:1  (local to host nodo1)
              UUID : 1e09b2dd:22728281:4161e36f:543bd901
            Events : 39

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
~~~

### Redimensionar a RAID completo

Para poder redimensionar a el RAID completo, basta con ejecutar lo siguiente:

~~~
root@nodo1:/home/vagrant# growpart /dev/md1 1
CHANGED: partition=1 start=2048 old: size=1024000 end=1026048 new: size=2090975,end=2093023
~~~

Y comprobamos:

~~~
root@nodo1:/home/vagrant# lsblk
NAME      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda         8:0    0 19.8G  0 disk  
├─sda1      8:1    0 18.8G  0 part  /
├─sda2      8:2    0    1K  0 part  
└─sda5      8:5    0 1021M  0 part  [SWAP]
sdb         8:16   0    1G  0 disk  
└─md1       9:1    0 1022M  0 raid1 
  └─md1p1 259:1    0 1021M  0 part  /mnt/raid1
sdc         8:32   0    1G  0 disk  
└─md1       9:1    0 1022M  0 raid1 
  └─md1p1 259:1    0 1021M  0 part  /mnt/raid1
sdd         8:48   0    1G  0 disk  
└─md1       9:1    0 1022M  0 raid1 
  └─md1p1 259:1    0 1021M  0 part  /mnt/raid1
sde         8:64   0    1G  0 disk  
~~~

Y ejecutamos lo siguiente para redimensionar el sistema de archivos:

~~~
	root@nodo1:/home/vagrant# resize2fs /dev/md1p1 
resize2fs 1.44.5 (15-Dec-2018)
Filesystem at /dev/md1p1 is mounted on /mnt/raid1; on-line resizing required
old_desc_blocks = 2, new_desc_blocks = 4
The filesystem on /dev/md1p1 is now 1045484 (1k) blocks long.

root@nodo1:/home/vagrant# df -h /mnt/raid1/
Filesystem      Size  Used Avail Use% Mounted on
/dev/md1p1      981M  2.8M  933M   1% /mnt/raid1
~~~