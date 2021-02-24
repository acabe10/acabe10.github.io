---
layout: post
title: Creación escenario OpenStack
tags: [all, OpenStack, Toboso]
---
# Introducción

Buenas, en este post vamos a crear un escenario en OpenStack el cuál vamos a usar durante el resto del curso para prácticar las tareas que vayamos teniendo. Va a constar inicialmente de 3 instancias con nombres relacionados con el libro "Don Quijote de la Mancha". Podremos seguir estas tareas siguiendo el tag <code>Toboso</code> al final del post.

![escenario](/assets/img/posts/toboso-openstack/escenario.png)

Pasos a realizar:

1. Creación de la red interna:
	* Nombre red interna de <nombre de usuario>
	* 10.0.1.0/24
2. Creación de las instancias
	* Dulcinea:
		* Debian Buster sobre volumen de 10GB con sabor m1.mini
		* Accesible directamente a través de la red externa y con una IP flotante
		* Conectada a la red interna, de la que será la puerta de enlace
	* Sancho:
		* Ubuntu 20.04 sobre volumen de 10GB con sabor m1.mini
		* Conectada a la red interna
		* Accesible indirectamente a través de dulcinea
	* Quijote:
		* CentOS 7 sobre volumen de 10GB con sabor m1.mini
		* Conectada a la red interna
		* Accesible indirectamente a través de dulcinea
3. Configuración de NAT en Dulcinea (Es necesario deshabilitar la seguridad en todos los puertos de dulcinea).
4. Definición de contraseña en todas las instancias (para poder modificarla desde consola en caso necesario).
5. Modificación de las instancias sancho y quijote para que usen direccionamiento estático y dulcinea como puerta de enlace.
6. Modificación de la subred de la red interna, deshabilitando el servidor DHCP.
7. Utilización de ssh-agent para acceder a las instancias.
8. Creación del usuario profesor en todas las instancias. Usuario que puede utilizar sudo sin contraseña.
9. Copia de las claves públicas de todos los profesores en las instancias para que puedan acceder con el usuario profesor.
10. Realiza una actualización completa de todos los servidores.
11. Configura el servidor con el nombre de dominio <nombre-usuario>.gonzalonazareno.org.
12. Hasta que no esté configurado el servidor DNS, incluye resolución estática en las tres instancias tanto usando nombre completo como hostname.
13. Asegúrate que el servidor tiene sincronizado su reloj utilizando un servidor NTP externo.

## Puntos a tener en cuenta al crear la red

Al crear la subred, en principio tenemos que dejar la puerta de enlace en blanco, ya que puede dar fallos más tarde. Sí tenemos que activar el servidor DHCP hasta que no le pongamos IP estática a nuestras máquinas. 

## Creación de instancias

Hemos creado las instancias mediante volúmenes como mostramos en la siguiente imagen:

![1](/assets/img/posts/toboso-openstack/1.png)

Y las hemos asociado a la red correspondiente.

## NAT en dulcinea

Para hacer este paso nos hemos tenido que conectar mediante la API de OpenStack, para que os hagáis una idea, tenemos una VPN entre nuestra casa y el centro, donde tenemos montado OpenStack.

Hemos deshabilitado la seguridad en todos los puertos de <code>dulcinea</code>:

~~~
(openstackclient) ale@arya:~$ openstack server list
+--------------------------------------+-----------+--------+--------------------------------------------------------------------------------------------+--------------------------+----------+
| ID                                   | Name      | Status | Networks                                                                                   | Image                    | Flavor   |
+--------------------------------------+-----------+--------+--------------------------------------------------------------------------------------------+--------------------------+----------+
| e2df02f1-8478-42b3-87ef-372fd80f045a | quijote   | ACTIVE | red interna de alejandro.cabeza=10.0.1.3                                                   | N/A (booted from volume) | m1.mini  |
| 6faf1705-865b-48a8-a4b4-078513f7efb7 | sancho    | ACTIVE | red interna de alejandro.cabeza=10.0.1.10                                                  | N/A (booted from volume) | m1.mini  |
| 6d8ccd55-1f92-4e93-8785-b45a1a106b50 | dulcinea  | ACTIVE | red de alejandro.cabeza=10.0.0.8, 172.22.200.123; red interna de alejandro.cabeza=10.0.1.4 | N/A (booted from volume) | m1.mini  |
| 4899770e-bf5e-45a5-a2ce-1ae1cfa750c1 | web_flask | ACTIVE | red de alejandro.cabeza=10.0.0.4, 172.22.200.130                                           | Debian Buster 10.6       | ssd.mini |
+--------------------------------------+-----------+--------+--------------------------------------------------------------------------------------------+--------------------------+----------+
~~~
~~~
(openstackclient) ale@arya:~$ openstack port list
+--------------------------------------+------+-------------------+--------------------------------------------------------------------------+--------+
| ID                                   | Name | MAC Address       | Fixed IP Addresses                                                       | Status |
+--------------------------------------+------+-------------------+--------------------------------------------------------------------------+--------+
| 0056630d-97d1-48a0-8de0-f73452b4ae0b |      | fa:16:3e:af:1b:68 | ip_address='10.0.1.4', subnet_id='9beb57a4-2b20-480b-9687-7acfd0050282'  | ACTIVE |
| 1e2a4c7a-91db-4ddd-a0ec-0ee5759b7031 |      | fa:16:3e:ef:10:6c | ip_address='10.0.0.2', subnet_id='63d2b1bb-4d50-48c9-8882-ea4dd5c4b202'  | ACTIVE |
| 27ac2863-98da-4662-9e8d-02771e99a9d8 |      | fa:16:3e:f6:c2:cf | ip_address='10.0.0.1', subnet_id='63d2b1bb-4d50-48c9-8882-ea4dd5c4b202'  | ACTIVE |
| 78040cd4-6348-4b2a-9f1a-5f789f387afe |      | fa:16:3e:81:e6:62 | ip_address='10.0.0.4', subnet_id='63d2b1bb-4d50-48c9-8882-ea4dd5c4b202'  | ACTIVE |
| a10cd1d5-6a1e-4963-ad36-4602a23ad613 |      | fa:16:3e:85:2e:e2 | ip_address='10.0.1.3', subnet_id='9beb57a4-2b20-480b-9687-7acfd0050282'  | ACTIVE |
| cca919c8-25b7-4709-9bca-57b498ae523f |      | fa:16:3e:38:51:46 | ip_address='10.0.0.8', subnet_id='63d2b1bb-4d50-48c9-8882-ea4dd5c4b202'  | ACTIVE |
| f23d7a78-4cac-41bd-ae39-5e01d8d11fe0 |      | fa:16:3e:9f:80:90 | ip_address='10.0.1.10', subnet_id='9beb57a4-2b20-480b-9687-7acfd0050282' | ACTIVE |
+--------------------------------------+------+-------------------+--------------------------------------------------------------------------+--------+
~~~

Y eliminamos seleccionando el ID del puerto:

~~~
openstack server remove security group dulcinea default

openstack port set --disable-port-security cca919c8-25b7-4709-9bca-57b498ae523f

openstack port set --disable-port-security 0056630d-97d1-48a0-8de0-f73452b4ae0b
~~~

Luego hemos habilitado el bit de forwarding en <code>dulcinea</code> editando el fichero:

~~~
sudo nano /etc/sysctl.conf
~~~

Descomentamos la siguiente línea:

~~~
net.ipv4.ip_forward=1
~~~

Cargamos los datos guardados:

~~~
sudo sysctl -p /etc/sysctl.conf
~~~

También hemos añadido un script en <code>/usr/bin/</code> para que nos habilite la regla de <code>iptables</code> con el siguiente contenido:

~~~
#!/bin/sh

iptables -t nat -A POSTROUTING -s 10.0.1.0/24 -o eth0 -j SNAT --to 10.0.0.8
~~~

~~~
sudo chmod 500 /usr/bin/snat.sh
~~~

Y lo hemos activado en <code>systemd</code> de la siguiente forma:

~~~
sudo nano /etc/systemd/system/snat.service

[Unit]
Description=iptables boot
After=networking.service
StartLimitIntervalSec=0

[Service]
Type=oneshot
RemainAfterExit=True
User=root
ExecStart=/usr/bin/snat.sh

[Install]
WantedBy=multi-user.target
~~~

~~~
sudo systemctl start snat
sudo systemctl enable snat
~~~

## Contraseña en todas las instancias

Para todas hemos usado el mismo comando:

~~~
sudo passwd <usuario>
~~~

## Direccionamiento estático

### dulcinea

Accedemos al fichero de configuración:

~~~
sudo nano /etc/network/interfaces
~~~

Y en la interfaz de la red interna:

~~~
auto eth1
iface eth1 inet static
	address 10.0.1.4
	netmask 255.255.255.0
	network 10.0.1.0
~~~
Y comentamos la línea que dice:

~~~
source /etc/network/interfaces.d/*
~~~

Si no en cada reinicio tomará los valores de ese fichero.

### sancho

Accedemos al fichero de configuración de red:

~~~
sudo nano /etc/netplan/50-cloud-init.yaml
~~~

Y lo dejamos de la siguiente forma:

~~~
network:
    version: 2
    ethernets:
        ens3:
            dhcp4: no
            addresses: [10.0.1.5/24]
            gateway4: 10.0.1.10
            match:
                macaddress: fa:16:3e:1a:a4:b0
            mtu: 8950
            set-name: ens3
            nameservers:
                addresses: [192.168.202.2]
~~~

Y como nos dice arriba del fichero, tendremos que crear el siguiente para que no se nos pierda al reinicio por tema de <code>cloud-init</code>:

~~~
echo "network: {config: disabled}" > /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
~~~

### quijote

Accedemos al fichero de configuración:

~~~
sudo nano /etc/sysconfig/network-scripts/ifcfg-eth0
~~~

Y lo dejamos de la siguiente forma:

~~~
BOOTPROTO=static
DEVICE=eth0
HWADDR=fa:16:3e:8c:14:3f
MTU=8950
ONBOOT=yes
TYPE=Ethernet
USERCTL=no
IPADDR=10.0.1.9
PREFIX=24
GATEWAY=10.0.1.10
DNS1=192.168.202.2
DNS2=8.8.8.8
~~~

## Deshabilitamos servidor DHCP

En OpenStack, deshabilitamos el DHCP de la red:

![2](/assets/img/posts/toboso-openstack/2.png)

## SSH-AGENT para acceder a las instancias

Para poder usar <code>ssh-agent</code>, que normalmente viene activado en las sesiones debian, probamos a ejecutar en nuestra máquina anfitriona:

~~~
ssh-add -L
~~~

Lo cuál nos mostrará las claves que tenemos en él, si no tenemos ninguna, metemos la nuestra:

~~~
ssh-add .ssh/cloud.key
~~~

Y en el fichero de configuración de <code>ssh</code>:

~~~
nano /etc/ssh/ssh_config
~~~

Descomentamos la siguiente línea y la ponemos en <code>yes</code>:

~~~
ForwardAgent yes
~~~

Y si comprobamos el acceso:

~~~
ale@arya:~$ ssh debian@dulcinea
Linux dulcinea 4.19.0-12-cloud-amd64 #1 SMP Debian 4.19.152-1 (2020-10-18) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Nov 11 17:46:41 2020 from 172.23.0.10
debian@dulcinea:~$ ssh ubuntu@sancho
Welcome to Ubuntu 20.04.1 LTS (GNU/Linux 5.4.0-53-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Nov 11 19:36:01 CET 2020

  System load:  0.08              Processes:             103
  Usage of /:   18.6% of 9.52GB   Users logged in:       1
  Memory usage: 39%               IPv4 address for ens3: 10.0.1.10
  Swap usage:   0%

 * Introducing self-healing high availability clustering for MicroK8s!
   Super simple, hardened and opinionated Kubernetes for production.

     https://microk8s.io/high-availability

0 updates can be installed immediately.
0 of these updates are security updates.


Last login: Wed Nov 11 17:41:16 2020 from 10.0.1.4
ubuntu@sancho:~$
~~~

Y el acceso a <code>quijote</code>:

~~~
debian@dulcinea:~$ ssh centos@quijote
Last login: Wed Nov 11 18:04:27 2020 from dulcinea
[centos@quijote ~]$
~~~

## Creamos usuario profesor

Para crear el usuario, en las 3 máquinas hemos usado:

~~~
adduser profesor
~~~

También hemos creado la carpeta para guardar las claves de los profesores:

~~~
mkdir /home/profesor/.ssh
~~~

Y las metemos en el fichero correspondiente:

~~~
nano /home/profesor/.ssh/authorized_keys
~~~

También cambiamos los permisos de dicha carpeta:

~~~
chown -R profesor. /home/profesor/.ssh/
~~~

## Añadimos a sudoers

### dulcinea

Para que use sudo sin contraseña, editamos el fichero de cloud-init para que se nos haga al inicio en caso de reinicio:

~~~
sudo nano /etc/sudoers.d/debian-cloud-init
~~~

Y le añadimos la siguiente línea:

~~~
profesor ALL = NOPASSWD: ALL
~~~

### sancho

Editamos el fichero de <code>cloud-init</code> también como en debian:

~~~
sudo nano /etc/sudoers.d/90-cloud-init-users
~~~

Y añadimos la siguiente línea:

~~~
profesor ALL=(ALL) NOPASSWD:ALL
~~~

### quijote

Editamos el fichero:

~~~
sudo nano /etc/sudoers.d/90-cloud-init-users
~~~

Y añadimos la siguiente línea:

~~~
profesor ALL=(ALL) NOPASSWD:ALL
~~~

## Actualización servidores:

### dulcinea

~~~
sudo apt update && apt upgrade
~~~

### sancho

~~~
sudo apt update && apt upgrade
~~~

### quijote

~~~
sudo yum update
~~~

## Nombre dominio y resolución estática

### dulcinea

Tendremos que desactivar primero la línea que permite a <code>cloud-init</code> modificar el <code>/etc/hosts</code>:

~~~
sudo nano /etc/cloud/cloud.cfg
~~~

Y ponemos la siguiente línea en <code>false</code>:

~~~
manage_etc_hosts: false
~~~

Ahora ya podemos editar el <code>/etc/hosts</code> sin problemas:

~~~
sudo nano /etc/hosts
~~~

Y lo dejamos de la siguiente forma:

~~~
127.0.0.1 dulcinea.alejandro.cabeza.gonzalonazareno.org dulcinea
127.0.0.1 localhost

10.0.1.5 sancho.alejandro.cabeza.gonzalonazareno.org sancho
10.0.1.9 quijote.alejandro.cabeza.gonzalonazareno.org quijote
~~~

### sancho

Editamos el fichero:

~~~
sudo nano /etc/hosts
~~~

Y añadimos las siguientes líneas:

~~~
127.0.0.1 localhost
127.0.0.1 sancho.alejandro.cabeza.gonzalonazareno.org sancho

10.0.1.9 quijote.alejandro.cabeza.gonzalonazareno.org quijote
10.0.1.10 dulcinea.alejandro.cabeza.gonzalonazareno.org dulcinea
~~~

### quijote

Editamos el fichero para cambiar el hostname:

~~~
sudo nano /etc/hostname
~~~

Y lo dejamos así:

~~~
quijote
~~~

También cambiamos el fichero <code>/etc/hosts</code>:

~~~
sudo nano /etc/hosts
~~~

Y le añadimos lo siguiente:

~~~
127.0.0.1 quijote.alejandro.cabeza.gonzalonazareno.org quijote

10.0.1.10 dulcinea.alejandro.cabeza.gonzalonazareno.org	dulcinea
10.0.1.5 sancho.alejandro.cabeza.gonzalonazareno.org sancho
~~~

## NTP externo

### dulcinea y sancho

NTP nos da conflictos y viene instalado por defecto, asi que lo quitamos(en ubuntu puede que no venga instalado):

~~~
sudo apt purge ntp
~~~

Editamos el fichero:

~~~
nano /etc/systemd/timesyncd.conf
~~~

Y añadimos la siguiente línea en la cuál podemos poner el servidor NTP externo que más nos guste:

~~~
[Time]
NTP=es.pool.ntp.org
~~~

Seleccionamos nuestra zona horaria:

~~~
sudo timedatectl set-timezone Europe/Madrid
~~~

Reiniciamos el servicio:

~~~
sudo systemctl restart systemd-timesyncd.service 
~~~

Y activamos el ntp(en ubuntu puede que ya venga activado):

~~~
sudo timedatectl set-ntp true
~~~

Comprobamos que está funcionando:

~~~
timedatectl

Local time: Thu 2020-11-12 00:12:53 CET
time: Wed 2020-11-11 23:12:53 UTC
time: Wed 2020-11-11 23:12:54
zone: Europe/Madrid (CET, +0100)
System clock synchronized: yes
service: active
in local TZ: no
~~~

### quijote

Instalamos los paquetes necesarios:

~~~
sudo yum install ntp ntpdate
~~~

Editamos el fichero de configuración:

~~~
sudo nano /etc/ntp.conf
~~~

Y añadimos los servidores de nuestra preferencia:

~~~
server 0.es.pool.ntp.org
server 1.hora.rediris.es
server 2.hora.roa.es
~~~

Seleccionamos nuestra zona horaria:

~~~
sudo timedatectl set-timezone Europe/Madrid
~~~

Reiniciamos el servicio:

~~~
sudo systemctl restart ntpd
~~~

Comprobamos los servidores que está cogiendo:

~~~
[root@quijote ~]# ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
 dnscache-madrid 138.96.64.10     2 u    -   64    1  484.199    3.530   0.000
~~~

Y también comprobamos la sincronización:

~~~
timedatectl
      Local time: Thu 2020-11-12 00:31:42 CET
  Universal time: Wed 2020-11-11 23:31:42 UTC
        RTC time: Wed 2020-11-11 23:31:41
       Time zone: Europe/Madrid (CET, +0100)
     NTP enabled: yes
NTP synchronized: no
 RTC in local TZ: no
      DST active: no
 Last DST change: DST ended at
                  Sun 2020-10-25 02:59:59 CEST
                  Sun 2020-10-25 02:00:00 CET
 Next DST change: DST begins (the clock jumps one hour forward) at
                  Sun 2021-03-28 01:59:59 CET
                  Sun 2021-03-28 03:00:00 CEST
~~~