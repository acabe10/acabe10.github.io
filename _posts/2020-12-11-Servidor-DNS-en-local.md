---
layout: post
title: Servidor DNS
tags: [all, DNS, Servidor, DNSmasq, Bind9]
---
# Introducción

Buenas, en este post vamos a instalar un servidor DNS en nuestra red local, haremos primero la instalación de DNSmasq y posteriormente lo haremos con Bind9.

1. Tendremos un servidor web que sirve dos páginas web: `www.iesgn.org` y `departamentos.iesgn.org`
2. Vamos a instalar en nuestra red local un servidor DNS (lo puedes instalar en el mismo equipo que tiene el servidor web).
3. Voy a suponer en este documento que el nombre del servidor DNS va a ser cabezas.iesgn.org.

## Servidor DNSmasq

Instalamos `dnsmasq`:

~~~
sudo apt install dnsmasq
~~~

Lo paramos para modificarlo:

~~~
sudo systemctl stop dnsmasq
~~~

Editamos el archivo:

~~~
sudo nano /etc/dnsmasq.conf
~~~

Descomentamos la siguiente línea para que obtenga las direcciones de nuestro archivo `/etc/host`:

~~~
strict-order
~~~

Y añadimos las siguientes líneas:

~~~
interface=eth1

listen-address=192.168.100.13

listen-address=127.0.0.1
~~~

Si queremos que resuelva los nombres sin tener que añadirlo al `/etc/hosts`, lo tendremos que añadir en el fichero anterior de la siguiente forma:

~~~
address=/www.iesgn.org/192.168.100.13

address=/departamentos.iesgn.org/192.168.100.13
~~~

Añadimos al `/etc/hosts` lo siguiente:

~~~
192.168.200.2 www.pruebaetchost.org
~~~

Reiniciamos el servidor:

~~~
sudo systemctl restart dnsmasq
~~~

### Comprobamos en cliente

Mostramos fichero `/etc/hosts`:

~~~
127.0.1.1 cliente.novalocal cliente
127.0.0.1 localhost

# The following lines are desirable for IPv6 capable hosts
::1 ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts
~~~

Consulta a `www.iesgn.org`:

~~~
dig www.iesgn.org

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> www.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 22132
;; flags: qr aa rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.iesgn.org.			IN	A

;; ANSWER SECTION:
www.iesgn.org.		0	IN	A	192.168.100.5

;; Query time: 0 msec
;; SERVER: 192.168.100.13#53(192.168.100.5)
;; WHEN: Thu Nov 26 09:33:31 UTC 2020
;; MSG SIZE  rcvd: 58
~~~

Consulta a `www.josedomingo.org`:

~~~
dig www.josedomingo.org

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> www.josedomingo.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 3831
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.josedomingo.org.		IN	A

;; ANSWER SECTION:
www.josedomingo.org.	652	IN	CNAME	playerone.josedomingo.org.
playerone.josedomingo.org. 652	IN	A	137.74.161.90

;; Query time: 0 msec
;; SERVER: 192.168.100.13#53(192.168.100.5)
;; WHEN: Thu Nov 26 09:34:01 UTC 2020
;; MSG SIZE  rcvd: 103
~~~

Comprobamos resolución inversa:

~~~
dig -x 137.74.161.90

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> -x 137.74.161.90
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 23377
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 9dfa03658d5ea75f7ebc143f5fbf76c6892d8a8f7cfe290a (good)
;; QUESTION SECTION:
;90.161.74.137.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
90.161.74.137.in-addr.arpa. 84487 IN	PTR	playerone.josedomingo.org.

;; AUTHORITY SECTION:
161.74.137.in-addr.arpa. 170880	IN	NS	dns16.ovh.net.
161.74.137.in-addr.arpa. 170880	IN	NS	ns16.ovh.net.

;; Query time: 2 msec
;; SERVER: 192.168.100.13#53(192.168.100.5)
;; WHEN: Thu Nov 26 09:35:02 UTC 2020
;; MSG SIZE  rcvd: 168
~~~

Comprobamos `www.pruebaetchost.org`:

~~~
dig www.pruebaetchost.org

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> www.pruebaetchost.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 20298
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.pruebaetchost.org.		IN	A

;; ANSWER SECTION:
www.pruebaetchost.org.	0	IN	A	192.168.200.2

;; Query time: 0 msec
;; SERVER: 192.168.100.13#53(192.168.100.5)
;; WHEN: Thu Nov 26 11:40:33 UTC 2020
;; MSG SIZE  rcvd: 66
~~~

## Servidor bind9

### Instalación

En nuestro servidor instalamos bind9:

~~~
sudo apt install bind9
~~~

Y las recomendaciones:

~~~
sudo apt install bind9-doc dnsutils ufw python-ply-doc
~~~

### Ficheros que modificar

Para que funcione los nombres que queremos en nuestros clientes, vamos a modificar 3 archivos, el primero:

~~~
sudo nano /etc/bind/named.conf.local
~~~

Y lo dejamos de la siguiente forma, indicandole los fichero de resolución directa e inversa que usaremos:

~~~
include "/etc/bind/zones.rfc1918";

zone "iesgn.org" {
 type master;
 file "db.iesgn.org";
};

zone "100.168.192.in-addr.arpa" {
 type master;
 file "db.100.168.192";
};
~~~

Ahora tendremos que crear los ficheros definidos anteriormente:

~~~
sudo nano /var/cache/bind/db.iesgn.org
~~~

Y le añadimos el siguiente contenido:

~~~
$TTL 86400
@       IN      SOA     cabezas.iesgn.org. root.iesgn.org. (
                1       ; serial
                21600   ; refresh
                3600    ; retry
                604800  ; expire
                21600   ); minimum
;
@       IN      NS      cabezas.iesgn.org.
@       IN      MX  10  correo.iesgn.org.

$ORIGIN iesgn.org.

cabezas         IN      A       192.168.100.1
correo          IN      A       192.168.100.200
ftp             IN      A       192.168.100.201
www             IN      CNAME   cabezas
departamentos   IN      CNAME   cabezas
~~~

El fichero de resolución inversa:

~~~
sudo nano /var/cache/bind/db.100.168.192
~~~

Y añadimos lo siguiente:

~~~
$TTL 86400
@       IN      SOA     cabezas.iesgn.org. root.iesgn.org. (
                 1 ; serial
                 21600 ; refresh
                 3600 ; retry
                 604800 ; expire
                 21600 ); minimum
;
@       IN      NS      cabezas.iesgn.org.

$ORIGIN 100.168.192.in-addr.arpa.

1      IN      PTR     cabezas.iesgn.org.
200     IN      PTR     correo.iesgn.org.
201     IN      PTR     ftp.iesgn.org.
~~~

Reiniciamos el servicio:

~~~
sudo systemctl restart bind9
~~~

### Configuración en cliente

En el cliente modificamos el fichero:

~~~
sudo nano /etc/resolv.conf
~~~

Y añadimos lo siguiente:

~~~
nameserver 192.168.100.1
search iesgn.org
~~~

## Consultas

### Dirección de: cabezas.iesgn.org, www.iesgn.org, ftp.iesgn.org:

~~~
dig cabezas.iesgn.org

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> cabezas.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 61561
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 8924b12dce27651bdee9b1a85fc9f37a12e1ba3574a77fed (good)
;; QUESTION SECTION:
;cabezas.iesgn.org.		IN	A

;; ANSWER SECTION:
cabezas.iesgn.org.	86400	IN	A	192.168.100.7

;; AUTHORITY SECTION:
iesgn.org.		86400	IN	NS	cabezas.iesgn.org.

;; Query time: 1 msec
;; SERVER: 192.168.100.1#53(192.168.100.7)
;; WHEN: Fri Dec 04 08:29:45 UTC 2020
;; MSG SIZE  rcvd: 104
~~~
~~~
dig www.iesgn.org

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> www.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 36923
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 177693ec140789edb39eebff5fc9f3936ae6fa7b13b01114 (good)
;; QUESTION SECTION:
;www.iesgn.org.			IN	A

;; ANSWER SECTION:
www.iesgn.org.		86400	IN	CNAME	cabezas.iesgn.org.
cabezas.iesgn.org.	86400	IN	A	192.168.100.7

;; AUTHORITY SECTION:
iesgn.org.		86400	IN	NS	cabezas.iesgn.org.

;; Query time: 1 msec
;; SERVER: 192.168.100.1#53(192.168.100.7)
;; WHEN: Fri Dec 04 08:30:10 UTC 2020
;; MSG SIZE  rcvd: 122
~~~
~~~
dig ftp.iesgn.org

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> ftp.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 19729
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 21b3c8c492bd6cd21b743ef45fc9f3a365aa3e18bd928d63 (good)
;; QUESTION SECTION:
;ftp.iesgn.org.			IN	A

;; ANSWER SECTION:
ftp.iesgn.org.		86400	IN	A	192.168.100.201

;; AUTHORITY SECTION:
iesgn.org.		86400	IN	NS	cabezas.iesgn.org.

;; ADDITIONAL SECTION:
cabezas.iesgn.org.	86400	IN	A	192.168.100.7

;; Query time: 0 msec
;; SERVER: 192.168.100.1#53(192.168.100.7)
;; WHEN: Fri Dec 04 08:30:27 UTC 2020
;; MSG SIZE  rcvd: 124
~~~

### NS de iesgn.org

~~~
dig NS iesgn.org

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> NS iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 29233
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: f8e358930f47f8b71a9d148a5fc9f3b8a479fc17f742ced8 (good)
;; QUESTION SECTION:
;iesgn.org.			IN	NS

;; ANSWER SECTION:
iesgn.org.		86400	IN	NS	cabezas.iesgn.org.

;; ADDITIONAL SECTION:
cabezas.iesgn.org.	86400	IN	A	192.168.100.7

;; Query time: 1 msec
;; SERVER: 192.168.100.1#53(192.168.100.7)
;; WHEN: Fri Dec 04 08:30:48 UTC 2020
;; MSG SIZE  rcvd: 104
~~~

### Correo de iesgn.org

~~~
dig MX iesgn.org

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> MX iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64017
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: bec532415b6ca86cea0a0a995fc9f3c629d57f705b81a38e (good)
;; QUESTION SECTION:
;iesgn.org.			IN	MX

;; ANSWER SECTION:
iesgn.org.		86400	IN	MX	10 correo.iesgn.org.

;; AUTHORITY SECTION:
iesgn.org.		86400	IN	NS	cabezas.iesgn.org.

;; ADDITIONAL SECTION:
correo.iesgn.org.	86400	IN	A	192.168.100.200
cabezas.iesgn.org.	86400	IN	A	192.168.100.7

;; Query time: 1 msec
;; SERVER: 192.168.100.1#53(192.168.100.7)
;; WHEN: Fri Dec 04 08:31:02 UTC 2020
;; MSG SIZE  rcvd: 143
~~~

### www.josedomingo.org 

~~~
dig www.josedomingo.org

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> www.josedomingo.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2891
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 5, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: a990851c48716691861a48b95fc9f3d927ebf6aca77c95ad (good)
;; QUESTION SECTION:
;www.josedomingo.org.		IN	A

;; ANSWER SECTION:
www.josedomingo.org.	900	IN	CNAME	playerone.josedomingo.org.
playerone.josedomingo.org. 900	IN	A	137.74.161.90

;; AUTHORITY SECTION:
josedomingo.org.	86400	IN	NS	ns4.cdmondns-01.org.
josedomingo.org.	86400	IN	NS	ns2.cdmon.net.
josedomingo.org.	86400	IN	NS	ns1.cdmon.net.
josedomingo.org.	86400	IN	NS	ns5.cdmondns-01.com.
josedomingo.org.	86400	IN	NS	ns3.cdmon.net.

;; ADDITIONAL SECTION:
ns4.cdmondns-01.org.	86400	IN	A	52.58.66.183

;; Query time: 391 msec
;; SERVER: 192.168.100.1#53(192.168.100.7)
;; WHEN: Fri Dec 04 08:31:21 UTC 2020
;; MSG SIZE  rcvd: 258
~~~

### Resolución inversa

~~~
dig -x 192.168.100.201

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> -x 192.168.100.201
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 63992
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: a16b75dd98c161b5b12249c85fc9f43ea13184e7e5b0f7b2 (good)
;; QUESTION SECTION:
;201.100.168.192.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
201.100.168.192.in-addr.arpa. 86400 IN	PTR	ftp.iesgn.org.

;; AUTHORITY SECTION:
100.168.192.in-addr.arpa. 86400	IN	NS	cabezas.iesgn.org.

;; ADDITIONAL SECTION:
cabezas.iesgn.org.	86400	IN	A	192.168.100.7

;; Query time: 1 msec
;; SERVER: 192.168.100.1#53(192.168.100.7)
;; WHEN: Fri Dec 04 08:33:02 UTC 2020
;; MSG SIZE  rcvd: 150
~~~

## Instalación y configuración DNS esclavo

### Configuración maestro

En nuestro servidor maestro, no permitimos la transferencia de archivos a todos los servidores, para ello:

~~~
sudo nano /etc/bind/named.conf.options
~~~

Y añadimos la siguiente línea:

~~~
allow-transfer { none; };
~~~

También tenemos que añadir en los ficheros de configuración:

~~~
sudo nano /etc/bind/named.conf.local
~~~

Añadiendo para que quede de la siguiente forma:

~~~
include "/etc/bind/zones.rfc1918";

zone "iesgn.org" {
 type master;
 file "db.iesgn.org";
 allow-transfer {192.168.100.3;};
 notify yes;
};

zone "100.168.192.in-addr.arpa" {
 type master;
 file "db.100.168.192";
 allow-transfer {192.168.100.3;};
 notify yes;
};
~~~

Y también en el fichero de zona directa:

~~~
sudo nano /var/cache/bind/db.iesgn.org
~~~

Lo dejamos de la siguiente forma:

~~~
$TTL 86400
@       IN      SOA     cabezas.iesgn.org. root.iesgn.org. (
                2       ; serial
                21600   ; refresh
                3600    ; retry
                604800  ; expire
                21600   ); minimum
;
@       IN      NS      cabezas.iesgn.org.
@       IN      NS      cabezas2.iesgn.org.
@       IN      MX  10  correo.iesgn.org.

$ORIGIN iesgn.org.

cabezas         IN      A       192.168.100.1
cabezas2       IN      A       192.168.100.3
correo          IN      A       192.168.100.200
ftp             IN      A       192.168.100.201
www             IN      CNAME   cabezas
departamentos   IN      CNAME   cabezas
~~~

También el de resolución inversa:

~~~
sudo nano /var/cache/bind/db.100.168.192
~~~

~~~
$TTL 86400
@       IN      SOA     cabezas.iesgn.org. root.iesgn.org. (
                 2 ; serial
                 21600 ; refresh
                 3600 ; retry
                 604800 ; expire
                 21600 ); minimum
;
@       IN      NS      cabezas.iesgn.org.
@       IN      NS      cabezas2.iesgn.org.

$ORIGIN 100.168.192.in-addr.arpa.

1      IN      PTR     cabezas.iesgn.org.
3   IN      PTR     cabezas2.iesgn.org.
200     IN      PTR     correo.iesgn.org.
201     IN      PTR     ftp.iesgn.org.
~~~

### Comprobación errores

Comprobamos si tenemos algún error en los ficheros:

~~~
sudo named-checkconf /etc/bind/named.conf.local
sudo named-checkzone iesgn.org /var/cache/bind/db.iesgn.org
sudo named-checkzone 100.168.192.in-addr.arpa /var/cache/bind/db.100.168.192
~~~

Reiniciamos el servicio:

~~~
systemctl restart bind9
~~~

### Instalación y configuración en esclavo

Hemos creado otra máquina en el cloud, llamada `cabezas-2`, vamos a instalar bind9:

~~~
sudo apt install bind9
~~~

Y las recomendaciones:

~~~
sudo apt install bind9-doc dnsutils ufw geoip-bin python-ply-doc
~~~

Y modificamos los archivos de configuración:

~~~
sudo nano /etc/bind/named.conf.local
~~~

Para dejarlo de la siguiente forma:

~~~
include "/etc/bind/zones.rfc1918";

zone "iesgn.org" {
 type slave;
 file "db.iesgn.org";
 masters { 192.168.100.1; };
};

zone "100.168.192.in-addr.arpa" {
 type slave;
 file "db.100.168.192";
 masters { 192.168.100.1; };
};
~~~

Reiniciamos el servicio:

~~~
sudo systemctl restart bind9
~~~

Mostramos la salida del comando donde vemos la transferencia:

~~~
sudo less /var/log/syslog
~~~

~~~
Dec  6 08:45:12 cabezas2 named[2882]: zone iesgn.org/IN: Transfer started.
Dec  6 08:45:12 cabezas2 named[2882]: network unreachable resolving './DNSKEY/IN': 2001:500:2::c#53
Dec  6 08:45:12 cabezas2 named[2882]: transfer of 'iesgn.org/IN' from 192.168.100.1#53: connected using 192.168.100.3#37907
Dec  6 08:45:12 cabezas2 named[2882]: network unreachable resolving './NS/IN': 2001:500:2::c#53
Dec  6 08:45:12 cabezas2 named[2882]: network unreachable resolving './DNSKEY/IN': 2001:500:9f::42#53
Dec  6 08:45:12 cabezas2 named[2882]: network unreachable resolving './NS/IN': 2001:500:9f::42#53
Dec  6 08:45:12 cabezas2 named[2882]: network unreachable resolving './DNSKEY/IN': 2001:503:c27::2:30#53
Dec  6 08:45:12 cabezas2 named[2882]: network unreachable resolving './NS/IN': 2001:503:c27::2:30#53
Dec  6 08:45:12 cabezas2 named[2882]: network unreachable resolving './DNSKEY/IN': 2001:500:1::53#53
Dec  6 08:45:12 cabezas2 named[2882]: zone iesgn.org/IN: transferred serial 2
Dec  6 08:45:12 cabezas2 named[2882]: transfer of 'iesgn.org/IN' from 192.168.100.1#53: Transfer status: success
Dec  6 08:45:12 cabezas2 named[2882]: transfer of 'iesgn.org/IN' from 192.168.100.1#53: Transfer completed: 1 messages, 9 records, 247 bytes, 0.001 secs (247000 bytes/sec)
Dec  6 08:45:12 cabezas2 named[2882]: network unreachable resolving './NS/IN': 2001:500:1::53#53
Dec  6 08:45:12 cabezas2 named[2882]: network unreachable resolving './DNSKEY/IN': 2001:500:12::d0d#53
Dec  6 08:45:12 cabezas2 named[2882]: network unreachable resolving './NS/IN': 2001:500:12::d0d#53
Dec  6 08:45:12 cabezas2 named[2882]: network unreachable resolving './DNSKEY/IN': 2001:7fd::1#53
Dec  6 08:45:12 cabezas2 named[2882]: network unreachable resolving './NS/IN': 2001:7fd::1#53
Dec  6 08:45:12 cabezas2 named[2882]: network unreachable resolving './DNSKEY/IN': 2001:500:2f::f#53
Dec  6 08:45:12 cabezas2 named[2882]: network unreachable resolving './NS/IN': 2001:500:2f::f#53
Dec  6 08:45:12 cabezas2 named[2882]: network unreachable resolving './DNSKEY/IN': 2001:7fe::53#53
Dec  6 08:45:12 cabezas2 named[2882]: network unreachable resolving './NS/IN': 2001:7fe::53#53
Dec  6 08:45:12 cabezas2 named[2882]: network unreachable resolving './DNSKEY/IN': 2001:500:200::b#53
Dec  6 08:45:12 cabezas2 named[2882]: network unreachable resolving './NS/IN': 2001:500:200::b#53
Dec  6 08:45:13 cabezas2 named[2882]: managed-keys-zone: Key 20326 for zone . acceptance timer complete: key now trusted
Dec  6 08:45:13 cabezas2 named[2882]: resolver priming query complete
Dec  6 08:45:13 cabezas2 named[2882]: zone 100.168.192.in-addr.arpa/IN: Transfer started.
Dec  6 08:45:13 cabezas2 named[2882]: transfer of '100.168.192.in-addr.arpa/IN' from 192.168.100.1#53: connected using 192.168.100.3#55911
Dec  6 08:45:13 cabezas2 named[2882]: zone 100.168.192.in-addr.arpa/IN: transferred serial 2
Dec  6 08:45:13 cabezas2 named[2882]: transfer of '100.168.192.in-addr.arpa/IN' from 192.168.100.1#53: Transfer status: success
Dec  6 08:45:13 cabezas2 named[2882]: transfer of '100.168.192.in-addr.arpa/IN' from 192.168.100.1#53: Transfer completed: 1 messages, 6 records, 213 bytes, 0.002 secs (106500 bytes/sec)
~~~

## Configuración cliente

Añadimos el servidor esclavo a su fichero:

~~~
sudo nano /etc/resolv.conf
~~~

Dejándolo de la siguiente forma:

~~~
nameserver 192.168.100.1
nameserver 192.168.100.3
search iesgn.org
~~~

Hacemos una consulta al maestro, nos fijamos en el apartado norec, en el bit AA y en que los dos servidores coinciden el número de serie al hacer la pregunta en los dos:

~~~
dig +norec @192.168.100.1 iesgn.org soa

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> +norec @192.168.100.1 iesgn.org soa
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 41648
;; flags: qr aa ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 61d33d595124c22a6cf263ba5fcc9d8c0887c88ee3cba93c (good)
;; QUESTION SECTION:
;iesgn.org.			IN	SOA

;; ANSWER SECTION:
iesgn.org.		86400	IN	SOA	cabezas.iesgn.org. root.iesgn.org. 2 21600 3600 604800 21600

;; AUTHORITY SECTION:
iesgn.org.		86400	IN	NS	cabezas.iesgn.org.
iesgn.org.		86400	IN	NS	cabezas2.iesgn.org.

;; ADDITIONAL SECTION:
cabezas.iesgn.org.	86400	IN	A	192.168.100.1
cabezas2.iesgn.org.	86400	IN	A	192.168.100.3

;; Query time: 0 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Sun Dec 06 08:59:56 GMT 2020
;; MSG SIZE  rcvd: 184

~~~

Hacemos la consulta al esclavo:

~~~
dig +norec @192.168.100.3 iesgn.org soa

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> +norec @192.168.100.3 iesgn.org soa
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9618
;; flags: qr aa ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 6df15f353e15108639ee49f85fcc9e122f1a244dbf859a1b (good)
;; QUESTION SECTION:
;iesgn.org.			IN	SOA

;; ANSWER SECTION:
iesgn.org.		86400	IN	SOA	cabezas.iesgn.org. root.iesgn.org. 2 21600 3600 604800 21600

;; AUTHORITY SECTION:
iesgn.org.		86400	IN	NS	cabezas.iesgn.org.
iesgn.org.		86400	IN	NS	cabezas2.iesgn.org.

;; ADDITIONAL SECTION:
cabezas.iesgn.org.	86400	IN	A	192.168.100.1
cabezas2.iesgn.org.	86400	IN	A	192.168.100.3

;; Query time: 0 msec
;; SERVER: 192.168.100.3#53(192.168.100.3)
;; WHEN: Sun Dec 06 09:02:10 GMT 2020
;; MSG SIZE  rcvd: 184
~~~

Y como podemos comprobar el número de serie coincide:

~~~
21600 3600 604800 21600
~~~

Ahora vamos a solicitar una copia desde el cliente para verificar que no funciona:

~~~
dig @192.168.100.1 iesgn.org axfr

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> @192.168.100.1 iesgn.org axfr
; (1 server found)
;; global options: +cmd
; Transfer failed.
~~~

Probamos desde el esclavo:

~~~
dig @192.168.100.1 iesgn.org axfr

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> @192.168.100.1 iesgn.org axfr
; (1 server found)
;; global options: +cmd
iesgn.org.		86400	IN	SOA	cabezas.iesgn.org. root.iesgn.org. 2 21600 3600 604800 21600
iesgn.org.		86400	IN	NS	cabezas.iesgn.org.
iesgn.org.		86400	IN	NS	cabezas2.iesgn.org.
iesgn.org.		86400	IN	MX	10 correo.iesgn.org.
cabezas.iesgn.org.	86400	IN	A	192.168.100.1
cabezas2.iesgn.org.	86400	IN	A	192.168.100.3
correo.iesgn.org.	86400	IN	A	192.168.100.200
departamentos.iesgn.org. 86400	IN	CNAME	cabezas.iesgn.org.
ftp.iesgn.org.		86400	IN	A	192.168.100.201
prueba.iesgn.org.	86400	IN	CNAME	cabezas.iesgn.org.
www.iesgn.org.		86400	IN	CNAME	cabezas.iesgn.org.
iesgn.org.		86400	IN	SOA	cabezas.iesgn.org. root.iesgn.org. 2 21600 3600 604800 21600
;; Query time: 0 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Sun Dec 06 09:09:48 GMT 2020
;; XFR size: 12 records (messages 1, bytes 346)
~~~

## Consultas

Hacemos una consulta:

~~~
dig www.josedomingo.org

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> www.josedomingo.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9850
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 5, ADDITIONAL: 6

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 46bce49ed2cd678ace13ceb05fcc9e87f5f1e6758d277191 (good)
;; QUESTION SECTION:
;www.josedomingo.org.		IN	A

;; ANSWER SECTION:
www.josedomingo.org.	900	IN	CNAME	playerone.josedomingo.org.
playerone.josedomingo.org. 900	IN	A	137.74.161.90

;; AUTHORITY SECTION:
josedomingo.org.	86399	IN	NS	ns2.cdmon.net.
josedomingo.org.	86399	IN	NS	ns3.cdmon.net.
josedomingo.org.	86399	IN	NS	ns5.cdmondns-01.com.
josedomingo.org.	86399	IN	NS	ns1.cdmon.net.
josedomingo.org.	86399	IN	NS	ns4.cdmondns-01.org.

;; ADDITIONAL SECTION:
ns1.cdmon.net.		172800	IN	A	35.189.106.232
ns2.cdmon.net.		172800	IN	A	35.195.57.29
ns3.cdmon.net.		172800	IN	A	35.157.47.125
ns4.cdmondns-01.org.	86399	IN	A	52.58.66.183
ns5.cdmondns-01.com.	172800	IN	A	52.59.146.62

;; Query time: 1267 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Sun Dec 06 09:04:07 GMT 2020
;; MSG SIZE  rcvd: 322
~~~

Y si nos fijamos en el apartado SERVER nos muestra que está respondiendo el maestro.

Vamos a apagar el maestro para ver lo que ocurre:

~~~
sudo systemctl stop bind9
~~~

Hacemos otra vez la misma consulta:

~~~
dig www.josedomingo.org

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> www.josedomingo.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 13089
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 5, ADDITIONAL: 5

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 578996e08c17658968f61eff5fcc9ee66a032ee4f0282752 (good)
;; QUESTION SECTION:
;www.josedomingo.org.		IN	A

;; ANSWER SECTION:
www.josedomingo.org.	805	IN	CNAME	playerone.josedomingo.org.
playerone.josedomingo.org. 805	IN	A	137.74.161.90

;; AUTHORITY SECTION:
josedomingo.org.	86306	IN	NS	ns4.cdmondns-01.org.
josedomingo.org.	86306	IN	NS	ns2.cdmon.net.
josedomingo.org.	86306	IN	NS	ns1.cdmon.net.
josedomingo.org.	86306	IN	NS	ns3.cdmon.net.
josedomingo.org.	86306	IN	NS	ns5.cdmondns-01.com.

;; ADDITIONAL SECTION:
ns1.cdmon.net.		172706	IN	A	35.189.106.232
ns2.cdmon.net.		172706	IN	A	35.195.57.29
ns3.cdmon.net.		172706	IN	A	35.157.47.125
ns4.cdmondns-01.org.	86306	IN	A	52.58.66.183

;; Query time: 1 msec
;; SERVER: 192.168.100.3#53(192.168.100.3)
;; WHEN: Sun Dec 06 09:05:42 GMT 2020
;; MSG SIZE  rcvd: 306
~~~

Y comprobamos que está respondiendo el esclavo.

## Delegación de subdominios

Hemos creado una nueva máquina llamada `cabezas3` para delegar el subdominio `informatica.iesgn.org`, lo primero que tendremos que hacer es modificar el archivo de zona de nuestro servidor principal `cabezas`:

~~~
sudo nano /var/cache/bind/db.iesgn.org
~~~

Y añadimos lo siguiente, no haciendo falta añadir su ip ya que la tenemos definida anteriormente. No olvidamos cambiar su serial.

~~~
$ORIGIN informatica.iesgn.org.
@               IN      NS      cabezas3
cabezas3        IN      A       192.168.100.4
~~~

Reiniciamos servicio:

~~~
sudo systemctl restart bind9
~~~

Y ahora en `cabezas3`, previamente instalado `bind9`, añadimos una nueva zona:

~~~
sudo nano /etc/bind/named.conf.local
~~~

~~~
zone "informatica.iesgn.org" {
 type master;
 file "db.informatica.iesgn.org";
};
~~~

Y creamos el fichero de zona:

~~~
sudo nano /var/cache/bind/db.informatica.iesgn.org
~~~

~~~
$TTL 86400
@       IN      SOA     cabezas3.informatica.iesgn.org. root.informatica.iesgn.org. (
                1       ; serial
                21600   ; refresh
                3600    ; retry
                604800  ; expire
                21600   ); minimum
;
@       IN      NS      cabezas3.informatica.iesgn.org.
@       IN      MX 10   correo.informatica.iesgn.org.

$ORIGIN informatica.iesgn.org.

cabezas3        IN      A       192.168.100.4
www             IN      A       192.168.100.100
ftp             IN      CNAME   cabezas3
correo          IN      A       192.168.100.101
~~~

Reiniciamos el servicio:

~~~
sudo systemctl restart bind9
~~~

## Consultas de prueba

### www.informatica.iesgn.org

~~~
dig www.informatica.iesgn.org

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> www.informatica.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 48477
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 7b1af9c9c38549ec112030605fccc168e5aac01988c03caf (good)
;; QUESTION SECTION:
;www.informatica.iesgn.org.	IN	A

;; ANSWER SECTION:
www.informatica.iesgn.org. 85946 IN	A	192.168.100.100

;; AUTHORITY SECTION:
informatica.iesgn.org.	86400	IN	NS	cabezas3.informatica.iesgn.org.

;; Query time: 0 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Sun Dec 06 11:32:55 GMT 2020
;; MSG SIZE  rcvd: 121
~~~

### ftp.informatica.iesgn.org

~~~
dig ftp.informatica.iesgn.org

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> ftp.informatica.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 4883
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 63e14bd1849ceadcb2ff67125fccc186cd51e32796f8e9ee (good)
;; QUESTION SECTION:
;ftp.informatica.iesgn.org.	IN	A

;; ANSWER SECTION:
ftp.informatica.iesgn.org. 86400 IN	CNAME	cabezas3.informatica.iesgn.org.

;; AUTHORITY SECTION:
informatica.iesgn.org.	10800	IN	SOA	cabezas3.informatica.iesgn.org. root.informatica.iesgn.org. 1 21600 3600 604800 21600

;; Query time: 2 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Sun Dec 06 11:33:26 GMT 2020
;; MSG SIZE  rcvd: 155
~~~

### NS informatica.iesgn.org

~~~
dig NS informatica.iesgn.org

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> NS informatica.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7285
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: a4f3a19666293c74177c044a5fccc20cc95de8530d75d6f6 (good)
;; QUESTION SECTION:
;informatica.iesgn.org.		IN	NS

;; ANSWER SECTION:
informatica.iesgn.org.	86400	IN	NS	cabezas3.informatica.iesgn.org.

;; Query time: 1 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Sun Dec 06 11:35:40 GMT 2020
;; MSG SIZE  rcvd: 101
~~~

No es el mismo que su dominio principal:

~~~
dig NS iesgn.org

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> NS iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58691
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 4ff70d86ae981c1f28c4d80a5fccc220432f0c99585dd41b (good)
;; QUESTION SECTION:
;iesgn.org.			IN	NS

;; ANSWER SECTION:
iesgn.org.		86400	IN	NS	cabezas.iesgn.org.

;; ADDITIONAL SECTION:
cabezas.iesgn.org.	86400	IN	A	192.168.100.1

;; Query time: 0 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Sun Dec 06 11:35:59 GMT 2020
;; MSG SIZE  rcvd: 104
~~~

### Servidor correo para informatica.iesgn.org

~~~
dig MX informatica.iesgn.org

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> MX informatica.iesgn.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 53263
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: e5900aba8061a5e2a8c8d4f45fccc255712f94ee0d1b0bdb (good)
;; QUESTION SECTION:
;informatica.iesgn.org.		IN	MX

;; ANSWER SECTION:
informatica.iesgn.org.	86400	IN	MX	10 correo.informatica.iesgn.org.

;; AUTHORITY SECTION:
informatica.iesgn.org.	86327	IN	NS	cabezas3.informatica.iesgn.org.

;; Query time: 1 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Sun Dec 06 11:36:54 GMT 2020
;; MSG SIZE  rcvd: 124
~~~