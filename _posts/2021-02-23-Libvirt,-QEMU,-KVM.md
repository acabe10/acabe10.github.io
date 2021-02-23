---
layout: post
title: Libvirt/QEMU/KVM
tags: [all, Libvirt, QEMU, KVM, Migración, Virsh]
---
# Introducción

Buenas, en este post vamos a hacer una serie de ejercicios con libvirt, los cuáles al final del proceso migraremos una aplicación entre dos máquinas virtuales.

## Ejercicio 1

Crea con <code>virt-install</code> una imagen de Debian Buster con formato qcow2 y un tamaño máximo de 3GiB. Esta imagen se denominará <code>buster-base,qcow2</code>. El sistema de ficheros del sistema instalado
en esta imagen será XFS. La imagen debe estar configurada para
poder usar hasta dos interfaces de red por dhcp. El usuario <code>debian</code> con contraseña <code>debian</code> puede utilizar sudo sin contraseña.

Creamos la imagen con los parámetros deseados:

~~~
virt-install --connect qemu:///system \
--cdrom ~/debian-10.6.0-amd64-netinst.iso \
--disk size=3 \
--network bridge=virbr0 \
--name buster-base \
--memory 1024 \
--vcpus 1
~~~

Comenzará la instalación:

![1](/assets/img/posts/libvirt/1.png)

Ponemos el sistema de ficheros XFS:

![2](/assets/img/posts/libvirt/2.png)

Y una vez seguidos los pasos básicos de una instalación que obviamente no vamos a poner, comprobarmos que se ha instalado correctamente accediendo desde <code>virt-viewer</code>:

![3](/assets/img/posts/libvirt/3.png)

O desde <code>ssh</code> comprobando antes su ip:

~~~
virsh -c qemu:///system net-dhcp-leases default      
 Expiry Time           MAC address         Protocol   IP address           Hostname   Client ID or DUID
------------------------------------------------------------------------------------------------------------------------------------------------
 2020-11-22 12:47:13   52:54:00:2e:b1:af   ipv4       192.168.122.249/24   debian     ff:00:2e:b1:af:00:01:00:01:27:4c:ed:89:52:54:00:2e:b1:af
~~~

Y conectamos:

~~~
ale@arya  ~  ssh debian@192.168.122.249                                 
debian@192.168.122.249's password: 
Linux debian 4.19.0-12-amd64 #1 SMP Debian 4.19.152-1 (2020-10-18) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Nov 22 10:36:12 2020
debian@debian:~$ 
~~~

Añadimos en el archivo:

~~~
/etc/sudoers
~~~

La siguiente línea si no se encuentra:

~~~
debian     ALL=(ALL) NOPASSWD: ALL
~~~

Y añadimos en el archivo:

~~~
sudo nano /etc/network/interfaces
~~~

El siguiente contenido:

~~~
auto ens3
iface ens3 inet dhcp

auto ens9 
iface ens9 inet dhcp
~~~

## Ejercicio 2

Crea un par de claves ssh en formato ecdsa y sin frase de paso y
agrega la clave pública al usuario `debian`

Creamos el par de claves en nuestra máquina anfitriona:

~~~
ssh-keygen -t ecdsa
Generating public/private ecdsa key pair.
Enter file in which to save the key (/home/ale/.ssh/id_ecdsa): /home/ale/.ssh/hlc
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/ale/.ssh/hlc.
Your public key has been saved in /home/ale/.ssh/hlc.pub.
The key fingerprint is:
SHA256:eQUexj1idja5e6eOaGdWr/xKXXuNc+rNYXNpcU0wPqI ale@arya
The key's randomart image is:
+---[ECDSA 256]---+
|         .+. .o  |
|         o=oB. o |
|         o.+o+o .|
|         . o.. o.|
|        S E  . .+|
|         .  . ooO|
|             o+@*|
|          ..+++**|
|         ..+.oB++|
+----[SHA256]-----+
~~~

Pasamos la clave al host creado:

~~~
 ale@arya  ~  ssh-copy-id -i ~/.ssh/hlc debian@192.168.122.249
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/ale/.ssh/hlc.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
debian@192.168.122.249's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'debian@192.168.122.249'"
and check to make sure that only the key(s) you wanted were added.
~~~

Comprobamos el acceso:

~~~
 ale@arya  ~  ssh debian@192.168.122.249
Linux debian 4.19.0-12-amd64 #1 SMP Debian 4.19.152-1 (2020-10-18) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Nov 22 12:11:47 2020 from 192.168.122.1
debian@debian:~$
~~~

## Ejercicio 3

Apagamos la máquina:

~~~
virsh -c qemu:///system shutdown buster-base
~~~

Comprobamos el tamaño de la imagen:

~~~
sudo qemu-img info /var/lib/libvirt/images/buster-base.qcow2

image: /var/lib/libvirt/images/buster-base.qcow2
file format: qcow2
virtual size: 3.0G (3221225472 bytes)
disk size: 1.5G
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: true
    refcount bits: 16
    corrupt: false
~~~

Reducimos su tamaño:

~~~
sudo virt-sparsify --in-place /var/lib/libvirt/images/buster-base.qcow2
 100% ⟦▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒⟧ --:--
[   7,6] Trimming /dev/sda1
[   7,8] Sparsify in-place operation completed with no errors
~~~

Y volvemos a comprobar:

~~~
sudo qemu-img info /var/lib/libvirt/images/buster-base.qcow2        

image: /var/lib/libvirt/images/buster-base.qcow2
file format: qcow2
virtual size: 3.0G (3221225472 bytes)
disk size: 1.3G
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: true
    refcount bits: 16
    corrupt: false
~~~

Se ha reducido, pero hemos averiguado que existe otra opción que lo reduce todavía más:

~~~
sudo virt-sparsify --compress /var/lib/libvirt/images/buster-base.qcow2 /var/lib/libvirt/images/buster-base-2.qcow2

[   0.0] Create overlay file in /tmp to protect source disk
[   0.0] Examine source disk
[   3,8] Fill free space in /dev/sda1 with zero
[   5,9] Copy to destination and make sparse
[  60,9] Sparsify operation completed with no errors.
virt-sparsify: Before deleting the old disk, carefully check that the 
target disk boots and works correctly.
~~~

Comprobamos:

~~~
sudo qemu-img info /var/lib/libvirt/images/buster-base-2.qcow2

image: /var/lib/libvirt/images/buster-base-2.qcow2
file format: qcow2
virtual size: 3.0G (3221225472 bytes)
disk size: 464M
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
~~~

## Ejercicio 4

Ya que tenemos los servidores de OVH, la voy a subir a mi página http://www.iesgn02.es, para ello, vamos a cambiar primero el propietario de _buster-base.qcow2_:

~~~
sudo chown -R ale. /var/lib/libvirt/images/buster-base-2.qcow2
~~~

Y ahora nos la pasamos a nuestro servidor igual que la clave privada:

~~~
scp /var/lib/libvirt/images/buster-base-2.qcow2 ned.iesgn02.es:/home/debian
scp ~/.ssh/hlc ned.iesgn02.es:/home/debian
~~~

## Ejercicio 5.1

Vamos a obtener una imagen la cuál sólo obtenga los cambios realizados a partir de la imagen base, para ello:

~~~
sudo qemu-img create -f qcow2 -o backing_file=/var/lib/libvirt/images/buster-base-2.qcow2 /var/lib/libvirt/images/maquina1.qcow2 5G
~~~

Comprobamos que se ha creado y observamos su tamaño:

~~~
virsh -c qemu:///system vol-list default
                 
 Name                Path
----------------------------------------------------------------
 buster-base.qcow2   /var/lib/libvirt/images/buster-base.qcow2
 maquina1.qcow2      /var/lib/libvirt/images/maquina1.qcow2
~~~
~~~
sudo qemu-img info /var/lib/libvirt/images/maquina1.qcow2

image: /var/lib/libvirt/images/maquina1.qcow2
file format: qcow2
virtual size: 5.0G (3221225472 bytes)
disk size: 196K
cluster_size: 65536
backing file: /var/lib/libvirt/images/buster-base-2.qcow2
Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
~~~

## Ejercicio 5.2

Creamos el fichero:

~~~
nano intra.xml
~~~

Con el siguiente contenido:

~~~
<network>
  <name>intra</name>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='br1' stp='on' delay='0'/>
  <ip address='10.10.20.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='10.10.20.50' end='10.10.20.100'/>
    </dhcp>
  </ip>
</network>
~~~

Definimos y activamos la red:

~~~
virsh -c qemu:///system net-define intra.xml
virsh -c qemu:///system net-start intra
~~~

Comprobamos:

~~~
virsh -c qemu:///system net-list       
 Name    State    Autostart   Persistent
------------------------------------------
 intra   active   no          yes
~~~

## Ejercicio 5.3

Lanzamos la máquina:

~~~
virt-install --connect qemu:///system --name maquina1 \
--autostart \
--memory 1024 \
--network network=intra \
--vcpus 1 \
--disk /var/lib/libvirt/images/maquina1.qcow2,bus=sata \
--import
~~~

## Ejercicio 5.4

Creamos el volumen adicional de 1GiB:

~~~
virsh -c qemu:///system vol-create-as --format raw --name vol-raw --capacity 1GiB --pool default
~~~

## Ejercicio 5.5

Conectamos el volumen a la máquina:

~~~
virsh -c qemu:///system attach-disk maquina1 /var/lib/libvirt/images/vol-raw sdb
~~~

Comprobamos en la máquina:

~~~
debian@debian:~$ lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sda      8:0    0   3G  0 disk 
└─sda1   8:1    0   3G  0 part /
sdb      8:16   0   1G  0 disk 
~~~

Creamos un sistema de fichero XFS dentro de la máquina:

~~~
sudo mkfs.xfs /dev/sdb 

meta-data=/dev/sdb               isize=512    agcount=4, agsize=65536 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=0
data     =                       bsize=4096   blocks=262144, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
~~~

Y montamos en la carpeta que creamos:

~~~
sudo mkdir /var/lib/pgsql
sudo mount /dev/sdb /var/lib/pgsql/
~~~

~~~
lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sda      8:0    0   3G  0 disk 
└─sda1   8:1    0   3G  0 part /
sdb      8:16   0   1G  0 disk /var/lib/pgsql
~~~

Cambiamos el propietario y grupo del directorio:

~~~
sudo chown -R postgres /var/lib/pgsql/
~~~

## Ejercicio 5.6

Instalamos _PostgreSQL_:

~~~
sudo apt install postgresql
~~~

Si entramos en postgre veremos dónde está alojado el directorio de sus datos:

~~~
sudo -u postgres psql
psql (11.9 (Debian 11.9-0+deb10u1))
Digite «help» para obtener ayuda.

postgres=# SHOW data_directory;
       data_directory        
-----------------------------
 /var/lib/postgresql/11/main
(1 fila)
~~~

Para cambiarlo, primero paramos postgre para garantizar la integridad de los datos:

~~~
sudo systemctl stop postgresql
~~~

Ahora instalamos _rsync_ que usaremos para hacer la copia:

~~~
sudo apt install rsync
~~~

Y hacemos la copia copiando los permisos con -a:

~~~
sudo rsync -av /var/lib/postgresql /var/lib/pgsql
~~~

Cambiamos el nombre de la carpeta actual de postgre por si falla algo:

~~~
sudo mv /var/lib/postgresql/11/main /var/lib/postgresql/11/main.bak
~~~

Para apuntar a la nueva ubicación editamos:

~~~
sudo nano /etc/postgresql/11/main/postgresql.conf
~~~

Y editamos la línea de _data directory_ dejándola de la siguiente forma:

~~~
data_directory = '/var/lib/pgsql/postgresql/11/main'
~~~

Reiniciamos postgre:

~~~
sudo systemctl restart postgresql
~~~

Y comprobamos:

~~~
sudo -u postgres psql
psql (11.9 (Debian 11.9-0+deb10u1))
Digite «help» para obtener ayuda.

postgres=# SHOW data_directory;
          data_directory           
-----------------------------------
 /var/lib/pgsql/postgresql/11/main
(1 fila)
~~~

Ya podríamos borrar el directorio antiguo de postgres:

~~~
sudo rm -rf /var/lib/postgresql/11/main.bak
~~~

## Ejercicio 5.7

Accedemos al usuario _postgres_ con contraseña _postgres_:

~~~
debian@debian:~$ su - postgres
Contraseña: 
postgres@debian:~$
~~~

Creamos base de datos:

~~~
postgres@debian:~$ createdb prueba
~~~

Accedemos a para comprobar la base de datos:

~~~
postgres@debian:~$ psql
psql (11.9 (Debian 11.9-0+deb10u1))
Digite «help» para obtener ayuda.

postgres=# \c prueba
Ahora está conectado a la base de datos «prueba» con el usuario «postgres».
prueba=#
~~~

## Ejercicio 5.8

Para que la base de datos sea accesible desde el exterior, editamos:

~~~
sudo nano /etc/postgresql/11/main/postgresql.conf
~~~

Y descomentamos la línea _listen addresses_ y la dejamos de la siguiente forma:

~~~
listen_addresses = '*'
~~~

También tendremos que editar el siguiente fichero:

~~~
sudo nano /etc/postgresql/11/main/pg_hba.conf
~~~

Cambiando las _IPv4 local connections_:

~~~
# IPv4 local connections:
host    all             all             all            md5
~~~

Hemos creado al usuario externo para el acceso desde el exterior:

~~~
debian@debian:~$ su postgres

postgres@debian:/home/debian$ createuser -S -R -D -l externo
~~~

Y le hemos dado una contraseña:

~~~
postgres@debian:/home/debian$ psql
psql (11.9 (Debian 11.9-0+deb10u1))
Digite «help» para obtener ayuda.

postgres=# ALTER USER externo WITH ENCRYPTED PASSWORD 'externo';ALTER ROLE
ALTER ROLE
~~~

Reiniciamos postgres:

~~~
sudo systemctl restart postgres
~~~

Ahora accedemos desde nuestra máquina:

~~~
 ale@arya  ~  psql -h 10.10.20.50 -U externo -d prueba 
Contraseña para usuario externo: 
psql (11.9 (Debian 11.9-0+deb10u1))
conexión SSL (protocolo: TLSv1.3, cifrado: TLS_AES_256_GCM_SHA384, bits: 256, compresión: desactivado)
Digite «help» para obtener ayuda.

prueba=> 
~~~

No nos ha hecho falta añadir ninguna línea en iptables, pero en caso de que fuera necesario, bastaría con hacer lo siguiente en maquina1:

~~~
sudo iptables -A INPUT -s 10.10.20.1 -p tcp --dport 5432 -j ACCEPT
sudo iptables -A OUTPUT -d 10.10.20.1 -p tcp --dport 5432 -j ACCEPT
~~~

Y en el hipervisor:

~~~
iptables -t nat -A PREROUTING -p tcp --dport 5997 -j DNAT --to 10.10.20.64:5997
~~~

## Ejercicio 5.11

Creamos la imagen para maquina2:

~~~
sudo qemu-img create -f qcow2 -o backing_file=/var/lib/libvirt/images/buster-base-2.qcow2 /var/lib/libvirt/images/maquina2.qcow2 4G~~~

Y ahora añadimos la imagen a dicho volumen:

Comprobamos:

~~~
sudo qemu-img info /var/lib/libvirt/images/maquina2.qcow2

image: /var/lib/libvirt/images/maquina2.qcow2
file format: qcow2
virtual size: 4.0G (3221225472 bytes)
disk size: 196K
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
~~~

## Ejercicio 5.12

Lanzamos la máquina:

~~~
virt-install --connect qemu:///system --name maquina2 \
--autostart \
--memory 1024 \
--network network=intra \
--vcpus 1 \
--disk /var/lib/libvirt/images/maquina2.qcow2,bus=sata \
--import
~~~

Obtenemos su ip:

~~~
virsh -c qemu:///system domifaddr maquina2
 Name       MAC address          Protocol     Address
-------------------------------------------------------------------------------
 vnet1      52:54:00:ff:e3:49    ipv4         10.10.20.64/24
~~~

## Ejercicio 5.13

En máquina1:

~~~
sudo systemctl stop postgresql
sudo umount /dev/sdb
~~~

En el hipervisor quitamos el volumen de maquina1 y lo asociamos a maquina2:

~~~
virsh -c qemu:///system detach-disk maquina1 /var/lib/libvirt/images/vol-raw
virsh -c qemu:///system attach-disk maquina2 /var/lib/libvirt/images/vol-raw sdb
~~~

En máquina2 creamos el directorio y montamos el volumen:

~~~
sudo mkdir /var/lib/pgsql
sudo mount /dev/sdb /var/lib/pgsql/
~~~

## Ejercicio 5.14

Cambiamos permisos de los ficheros para poderlos pasar con usuario debian:

~~~
sudo chown debian. /etc/postgresql/11/main/postgres.conf
sudo chown debian. /etc/postgresql/11/main/pg_hba.conf
~~~

Lo envíamos por scp a maquina2:

~~~
scp /etc/postgresql/11/main/postgresql.conf debian@10.10.20.64:/home/debian
scp /etc/postgresql/11/main/pg_hba.conf debian@10.10.20.64:/home/debian
~~~

## Ejercicio 5.15

Instalamos a través de ssh:

~~~
ssh -t debian@10.10.20.64 "sudo apt-get install postgresql"
~~~

En maquina2, copiamos los archivos de configuración a su sitio determinado:

~~~
sudo cp pg_hba.conf /etc/postgresql/11/main/pg_hba.conf
sudo cp postgresql.conf /etc/postgresql/11/main/postgresql.conf
~~~

Comprobamos que funciona desde nuestra máquina:

~~~
ale@arya  ~  psql -h 10.10.20.64 -U externo -d prueba
Contraseña para usuario externo: 
psql (11.9 (Debian 11.9-0+deb10u1))
conexión SSL (protocolo: TLSv1.3, cifrado: TLS_AES_256_GCM_SHA384, bits: 256, compresión: desactivado)
Digite «help» para obtener ayuda.

prueba=> 
~~~

## Ejercicio 5.16

Hemos creado otro fichero xml con el siguiente contenido:

~~~
<network>
  <name>intra_2</name>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='br2' stp='on' delay='0'/>
  <ip address='10.10.50.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='10.10.50.50' end='10.10.50.100'/>
    </dhcp>
  </ip>
</network>
~~~

Definimos la red:

~~~
virsh -c qemu:///system net-define intra_2.xml                      
~~~

Y la arrancamos:

~~~
virsh -c qemu:///system net-start intra_2
~~~

La conectamos a maquina2:

~~~
virsh -c qemu:///system attach-interface --domain maquina2 --type network --source intra_2 --config --live
~~~

Y comprobamos:

~~~
virsh -c qemu:///system domiflist maquina2 
 Interface   Type      Source    Model     MAC
--------------------------------------------------------------
 vnet1       network   intra     e1000     52:54:00:ff:e3:49
 vnet2       network   intra_2   rtl8139   52:54:00:0f:d7:e3
~~~

~~~
virsh -c qemu:///system domifaddr maquina2 
 Name       MAC address          Protocol     Address
-------------------------------------------------------------------------------
 vnet1      52:54:00:ff:e3:49    ipv4         10.10.20.64/24
 vnet2      52:54:00:fa:a6:73    ipv4         10.10.50.54/24
~~~

Todo correcto, quitamos la interfaz de intra:

~~~
virsh -c qemu:///system detach-interface --domain maquina2 --type network --mac 52:54:00:ff:e3:49
~~~

## Ejercicio 5.17

Accedemos desde nuestro hipervisor:

~~~
 ale@arya  ~  psql -h 10.10.50.54 -U externo -d prueba
Contraseña para usuario externo: 
psql (11.9 (Debian 11.9-0+deb10u1))
conexión SSL (protocolo: TLSv1.3, cifrado: TLS_AES_256_GCM_SHA384, bits: 256, compresión: desactivado)
Digite «help» para obtener ayuda.

prueba=> 
~~~

Si tuvieramos que acceder desde el exterior, tendríamos que cambiar la regla de iptables:

~~~
iptables -t nat -A PREROUTING -p tcp --dport 5997 -j DNAT --to 10.10.50.54:5997
~~~

## Ejercicio 5.18

Apagamos máquina1:

~~~
virsh -c qemu:///system shutdown maquina1
~~~

En máquina2 la memoria máxima que puede tener de ram la tenemos en 1G, para modificarla tendremos que apagar la máquina:

~~~
virsh -c qemu:///system shutdown maquina2
~~~

Entramos en la configuración de la máquina:

~~~
virsh -c qemu:///system edit maquina2
~~~

Y cambiamos el valor _memory_:

~~~
<memory unit='KiB'>4194304</memory>
~~~

Iniciamos la máquina:

~~~
virsh -c qemu:///system start maquina2
~~~

Y ahora en caliente, como tenemos el _memballoon_ activo, podremos aumentar la ram:

~~~
virsh -c qemu:///system setmem maquina2 --size 2G --live
~~~

Comprobamos dentro de máquina2:

~~~
free -h
              total        used        free      shared  buff/cache   available
Mem:          1,9Gi        64Mi       1,7Gi       5,0Mi        65Mi       1,6Gi
Swap:            0B          0B          0B
~~~