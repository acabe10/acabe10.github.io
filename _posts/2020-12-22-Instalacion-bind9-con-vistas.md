---
layout: post
title: Instalación y configuración bind9 con vistas
tags: [all, OpenStack, Toboso, Bind9]
---
# Introducción

Buenas, en este post vamos a instalar y configurar un servidor DNS con bind9 usando vistas, ya que tenemos el [escenario creado anteriormente](https://acabe10.github.io/2021-02-22-Creacion-escenario-OpenStack/) en nuestro OpenStack 

# Instalación

Vamos a instalar el servidor DNS en freston. Instalamos `bind9` y sus recomendaciones:

~~~
sudo apt install bind9 -y && sudo apt install -y bind9-doc dnsutils ufw python-ply-doc
~~~

## Ficheros a modificar

### named.conf.local

Modificamos el fichero:

~~~
sudo nano /etc/bind/named.conf.local
~~~

Y le añadimos el siguiente contenido:

~~~
view interna {
    match-clients { 10.0.1.0/24; };
    allow-recursion { any; };

        zone "cabezas.gonzalonazareno.org"
        {
                type master;
                file "db.interna.cabezas.gonzalonazareno.org";
        };
        zone "1.0.10.in-addr.arpa"
        {
                type master;
                file "db.1.0.10";
        };
        include "/etc/bind/zones.rfc1918";
        include "/etc/bind/named.conf.default-zones";
};

view dmz {
    match-clients { 10.0.2.0/24; };
    allow-recursion { any; };

        zone "cabezas.gonzalonazareno.org"
        {
                type master;
                file "db.dmz.cabezas.gonzalonazareno.org";
        };
        zone "2.0.10.in-addr.arpa"
        {
                type master;
                file "db.2.0.10";
        };
        include "/etc/bind/zones.rfc1918";
        include "/etc/bind/named.conf.default-zones";
};

view externa {
    match-clients { 172.22.0.0/15; 192.168.202.2; };
    allow-recursion { any; };

        zone "cabezas.gonzalonazareno.org"
        {
                type master;
                file "db.externa.cabezas.gonzalonazareno.org";
        };
        include "/etc/bind/zones.rfc1918";
        include "/etc/bind/named.conf.default-zones";
};
~~~

### named.conf

También tenemos que comentar en el fichero:

~~~
sudo nano /etc/bind/named.conf
~~~

La siguiente línea:

~~~
//include "/etc/bind/named.conf.default-zones";
~~~

### named.conf.options

Y también añadir al fichero:

~~~
sudo nano /etc/bind/named.conf.options
~~~

El siguiente contenido para que funcionen las peticiones desde todos los sitios:

~~~
allow-query { 172.22.0.0/15;10.0.1.0/24;10.0.2.0/24;192.168.202.2; };
allow-recursion { any; };
allow-query-cache { any; };
~~~

Ahora tendremos que añadir los ficheros de zona directa e inversa para cada una de las zonas, excepto la zona externa el de resolución inversa, empezamos con el primero:

### db.interna.cabezas.gonzalonazareno.org

Abrimos el fichero:

~~~
sudo nano /var/cache/bind/db.interna.cabezas.gonzalonazareno.org
~~~

Y le añadimos el siguiente contenido:

~~~
$TTL 86400
@       IN      SOA     freston.cabezas.gonzalonazareno.org. root.cabezas.gonzalonazareno.org. (
                1       ; serial
                21600   ; refresh
                3600    ; retry
                604800  ; expire
                21600   ); minimun
;
@       IN      NS      freston.cabezas.gonzalonazareno.org.

$ORIGIN cabezas.gonzalonazareno.org.
dulcinea        IN      A       10.0.1.10
freston         IN      A       10.0.1.9
sancho          IN      A       10.0.1.12
quijote         IN      A       10.0.2.4
~~~

### db.1.0.10

Abrimos el fichero:

~~~
sudo nano /var/cache/bind/db.1.0.10
~~~

Y le añadimos el siguiente contenido:

~~~
$TTL 86400
@       IN      SOA     freston.cabezas.gonzalonazareno.org. root.cabezas.gonzalonazareno.org. (
                 1 ; serial
                 21600 ; refresh
                 3600 ; retry
                 604800 ; expire
                 21600 ); minimum
;
@       IN      NS      freston.cabezas.gonzalonazareno.org.

$ORIGIN 1.0.10.in-addr.arpa.

9       IN      PTR     freston.cabezas.gonzalonazareno.org
10      IN      PTR     dulcinea.cabezas.gonzalonazareno.org
12      IN      PTR     sancho.cabezas.gonzalonazareno.org
~~~

### db.dmz.cabezas.gonzalonazareno.org

Abrimos el fichero:

~~~
sudo nano /var/cache/bind/db.dmz.cabezas.gonzalonazareno.org
~~~

Y le añadimos el siguiente contenido:

~~~
$TTL 86400
@       IN      SOA     freston.cabezas.gonzalonazareno.org. root.cabezas.gonzalonazareno.org. (
                1       ; serial
                21600   ; refresh
                3600    ; retry
                604800  ; expire
                21600   ); minimum
;
@       IN      NS      freston.cabezas.gonzalonazareno.org.

$ORIGIN cabezas.gonzalonazareno.org.

quijote         IN      A       10.0.2.4
dulcinea        IN      A       10.0.2.5
freston         IN      A       10.0.1.9
sancho          IN      A       10.0.1.12
~~~

### db.2.0.10

Abrimos el fichero:

~~~
sudo nano /var/cache/bind/db.2.0.10
~~~

Y le añadimos el siguiente contenido:

~~~
$TTL 86400
@       IN      SOA     freston.cabezas.gonzalonazareno.org. root.cabezas.gonzalonazareno.org. (
                 1 ; serial
                 21600 ; refresh
                 3600 ; retry
                 604800 ; expire
                 21600 ); minimum
;
@       IN      NS      freston.cabezas.gonzalonazareno.org.

$ORIGIN 2.0.10.in-addr.arpa.

4       IN      PTR     sancho.cabezas.gonzalonazareno.org
5      IN      PTR     dulcinea.cabezas.gonzalonazareno.org
~~~

### db.externa.cabezas.gonzalonazareno.org

Creamos el archivo:

~~~
sudo nano /var/cache/bind/db.externa.cabezas.gonzalonazareno.org
~~~

Y añadimos el siguiente contenido:

~~~
$TTL 86400
@       IN      SOA     freston.cabezas.gonzalonazareno.org. root.cabezas.gonzalonazareno.org. (
                1       ; serial
                21600   ; refresh
                3600    ; retry
                604800  ; expire
                21600   ); minimum
;
@       IN      NS      freston.cabezas.gonzalonazareno.org.

$ORIGIN cabezas.gonzalonazareno.org.
dulcinea        IN      A       172.22.200.123
~~~

# Regla DNAT en Dulcinea

Tendríamos que añadir una nueva regla a dulcinea para que funcionasen las peticiones desde el exterior:

~~~
iptables -t nat -A PREROUTING -i eth2 -p tcp --dport 53 -j DNAT --to 10.0.1.9:53
~~~

La podríamos añadir al fichero ya previamente creado en `/usr/bin/dmz-snat.sh` para que funcionara al reiniciar el equipo.

También hemos deshabilitado los grupos de seguridad y la seguridad de todos los puertos en todas las instancias para que el escenario sea más realista, además que si no lo quitamos dulcinea no puede hacer consultas.

# Configuración máquinas

Para que los DNS sean permanentes aunque reiniciemos la máquina, vamos a hacer un par de modificaciones:

### En dulcinea y freston

Abrimos el fichero:

~~~
sudo nano /etc/resolvconf/resolv.conf.d/base
~~~

Y le añadimos el siguiente contenido:

~~~
nameserver 10.0.1.9
search cabezas.gonzalonazareno.org
domain cabezas.gonzalonazareno.org
~~~

### En sancho

Editamos el fichero:

~~~
sudo nano /etc/systemd/resolved.conf
~~~

Y añadimos lo siguiente:

~~~
[Resolve]
DNS=10.0.1.9
~~~

### En quijote

En quijote ya lo tenemos definido por Network-manager en nuestro fichero:

~~~
sudo nano /etc/sysconfig/network-scripts/ifcfg-eth0
~~~

Con el siguiente contenido:

~~~
DEVICE=eth0
HWADDR=fa:16:3e:07:09:da
MTU=8950
ONBOOT=yes
TYPE=Ethernet
USERCTL=no
IPADDR=10.0.2.4
NETMASK=255.255.255.0
GATEWAY=10.0.2.5
DNS1=10.0.1.9
SEARCH=cabezas.gonzalonazareno.org
~~~

# Comprobaciones

## NS de cabezas.gonzalonazareno.org

### Cliente interno

~~~
ubuntu@sancho:~$ dig ns cabezas.gonzalonazareno.org

; <<>> DiG 9.16.1-Ubuntu <<>> ns cabezas.gonzalonazareno.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32749
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 21149d9a7d341aff5bffb6055fdc7f8867296c25c15c2096 (good)
;; QUESTION SECTION:
;cabezas.gonzalonazareno.org.	IN	NS

;; ANSWER SECTION:
cabezas.gonzalonazareno.org. 86400 IN	NS	freston.cabezas.gonzalonazareno.org.

;; ADDITIONAL SECTION:
freston.cabezas.gonzalonazareno.org. 86400 IN A	10.0.1.9

;; Query time: 0 msec
;; SERVER: 10.0.1.9#53(10.0.1.9)
;; WHEN: Fri Dec 18 11:08:08 CET 2020
;; MSG SIZE  rcvd: 122
~~~

### Cliente externo

~~~
ale@arya  ~  dig ns cabezas.gonzalonazareno.org      

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> ns cabezas.gonzalonazareno.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17457
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: f86d8cf0876fca410434860b5fdc7fd14b79bbae7b09b768 (good)
;; QUESTION SECTION:
;cabezas.gonzalonazareno.org.	IN	NS

;; ANSWER SECTION:
cabezas.gonzalonazareno.org. 79352 IN	NS	dulcinea.cabezas.gonzalonazareno.org.

;; ADDITIONAL SECTION:
dulcinea.cabezas.gonzalonazareno.org. 86211 IN A 172.22.200.123

;; Query time: 0 msec
;; SERVER: 192.168.202.2#53(192.168.202.2)
;; WHEN: vie dic 18 11:09:21 CET 2020
;; MSG SIZE  rcvd: 123
~~~

## IP de dulcinea

### Cliente interno

~~~
ubuntu@sancho:~$ dig dulcinea.cabezas.gonzalonazareno.org

; <<>> DiG 9.16.1-Ubuntu <<>> dulcinea.cabezas.gonzalonazareno.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8149
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: b877092789f407d914028c185fdc8062f303d0d4ef46138a (good)
;; QUESTION SECTION:
;dulcinea.cabezas.gonzalonazareno.org. IN A

;; ANSWER SECTION:
dulcinea.cabezas.gonzalonazareno.org. 86400 IN A 10.0.1.10

;; AUTHORITY SECTION:
cabezas.gonzalonazareno.org. 86400 IN	NS	freston.cabezas.gonzalonazareno.org.

;; ADDITIONAL SECTION:
freston.cabezas.gonzalonazareno.org. 86400 IN A	10.0.1.9

;; Query time: 3 msec
;; SERVER: 10.0.1.9#53(10.0.1.9)
;; WHEN: Fri Dec 18 11:11:46 CET 2020
;; MSG SIZE  rcvd: 147
~~~

### Cliente externo

~~~
ale@arya  ~  dig dulcinea.cabezas.gonzalonazareno.org

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> dulcinea.cabezas.gonzalonazareno.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 20306
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: c46ed144790395a0f41965495fdc8071bc495fa4411bc169 (good)
;; QUESTION SECTION:
;dulcinea.cabezas.gonzalonazareno.org. IN A

;; ANSWER SECTION:
dulcinea.cabezas.gonzalonazareno.org. 86051 IN A 172.22.200.123

;; AUTHORITY SECTION:
cabezas.gonzalonazareno.org. 79192 IN	NS	dulcinea.cabezas.gonzalonazareno.org.

;; Query time: 0 msec
;; SERVER: 192.168.202.2#53(192.168.202.2)
;; WHEN: vie dic 18 11:12:01 CET 2020
;; MSG SIZE  rcvd: 123
~~~

## Resolución de www

### Cliente interno

~~~
dig www.cabezas.gonzalonazareno.org

; <<>> DiG 9.16.1-Ubuntu <<>> www.cabezas.gonzalonazareno.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32933
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;www.cabezas.gonzalonazareno.org. IN	A

;; ANSWER SECTION:
www.cabezas.gonzalonazareno.org. 86400 IN CNAME	quijote.cabezas.gonzalonazareno.org.
quijote.cabezas.gonzalonazareno.org. 7199 IN A	10.0.2.4

;; Query time: 3 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Fri Dec 18 17:03:37 CET 2020
;; MSG SIZE  rcvd: 98
~~~

### Cliente externo

~~~
dig www.cabezas.gonzalonazareno.org

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> www.cabezas.gonzalonazareno.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 60029
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 1c378297163a972c1561e8445fdcd312e5ed16773f24941c (good)
;; QUESTION SECTION:
;www.cabezas.gonzalonazareno.org. IN	A

;; ANSWER SECTION:
www.cabezas.gonzalonazareno.org. 84389 IN CNAME	dulcinea.cabezas.gonzalonazareno.org.
dulcinea.cabezas.gonzalonazareno.org. 64898 IN A 172.22.200.123

;; AUTHORITY SECTION:
cabezas.gonzalonazareno.org. 58039 IN	NS	dulcinea.cabezas.gonzalonazareno.org.

;; Query time: 1 msec
;; SERVER: 192.168.202.2#53(192.168.202.2)
;; WHEN: Fri Dec 18 16:04:35 UTC 2020
;; MSG SIZE  rcvd: 141
~~~

## Resolución inversa

### Desde red interna

~~~
ubuntu@sancho:~$ dig -x 10.0.1.10

; <<>> DiG 9.16.1-Ubuntu <<>> -x 10.0.1.10
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 4636
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: e6f8ca703ce0063c1d25d25c5fdc80a15a56cf163ddd3294 (good)
;; QUESTION SECTION:
;10.1.0.10.in-addr.arpa.		IN	PTR

;; ANSWER SECTION:
10.1.0.10.in-addr.arpa.	86400	IN	PTR	dulcinea.cabezas.gonzalonazareno.org.1.0.10.in-addr.arpa.

;; AUTHORITY SECTION:
1.0.10.in-addr.arpa.	86400	IN	NS	freston.cabezas.gonzalonazareno.org.

;; ADDITIONAL SECTION:
freston.cabezas.gonzalonazareno.org. 86400 IN A	10.0.1.9

;; Query time: 0 msec
;; SERVER: 10.0.1.9#53(10.0.1.9)
;; WHEN: Fri Dec 18 11:12:49 CET 2020
;; MSG SIZE  rcvd: 195
~~~

### Desde DMZ

~~~
[centos@quijote ~]$ dig -x 10.0.2.5

; <<>> DiG 9.11.20-RedHat-9.11.20-5.el8 <<>> -x 10.0.2.5
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 42468
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: cfdc8b42c7d342dcd647c8f95fdc80c4f09bd5984b42b9bc (good)
;; QUESTION SECTION:
;5.2.0.10.in-addr.arpa.		IN	PTR

;; ANSWER SECTION:
5.2.0.10.in-addr.arpa.	86400	IN	PTR	dulcinea.cabezas.gonzalonazareno.org.2.0.10.in-addr.arpa.

;; AUTHORITY SECTION:
2.0.10.in-addr.arpa.	86400	IN	NS	freston.cabezas.gonzalonazareno.org.

;; ADDITIONAL SECTION:
freston.cabezas.gonzalonazareno.org. 86400 IN A	10.0.1.9

;; Query time: 1 msec
;; SERVER: 10.0.1.9#53(10.0.1.9)
;; WHEN: Fri Dec 18 11:13:24 CET 2020
;; MSG SIZE  rcvd: 194
~~~