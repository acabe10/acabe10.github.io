---
layout: post
title: Instalación Squid3 en Debian Buster
tags: [all, squid3, proxy, Debian, proxy inverso]
---
# Introducción

Buenas, en este post vamos a instalar un proxy `squid` para configurar nuestro cliente para que pueda acceder a internet por medio de este proxy.

Vamos a usar el siguiente fichero `Vagrantfile` para crear el escenario: un ordenador llamado `proxy` donde instalaremos `squid` y un cliente interno llamado `cliente_int`:

~~~
# -*- mode: ruby -*-
# vi: set ft=ruby :


Vagrant.configure("2") do |config|
config.vm.define :proxy do |proxy|
    proxy.vm.box = "debian/buster64"
    proxy.vm.hostname = "proxy"
    proxy.vm.network :private_network, ip: "10.0.0.10", virtualbox__intnet: "red_privada1"
    proxy.vm.network :private_network, ip: "192.168.200.10"
  end
  config.vm.define :cliente_int do |cliente_int|
    cliente_int.vm.box = "debian/buster64"
    cliente_int.vm.network :private_network, ip: "10.0.0.11",virtualbox__intnet: "red_privada1"
  end
  
end
~~~

## Instalación de squid3

Para instalar `squid3` ejecutaremos el siguiente comando:

~~~
sudo apt update && apt upgrade && apt install squid3
~~~

Editamos el fichero de configuración de `squid` para permitir las conexiones desde nuestro ordenador:

~~~
sudo nano /etc/squid/squid.conf
~~~

Hemos configurado uno nuevo ya que el que viene por defecto es demasiado largo:

~~~
acl miordenador src 192.168.200.0/24

acl SSL_ports port 443
acl Safe_ports port 80
acl Safe_ports port 443
acl Safe_ports port 21
acl CONNECT method CONNECT

http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
http_access allow localhost manager
http_access deny manager

http_access allow miordenador

http_access deny all

http_port 3128

coredump_dir /var/spool/squid
~~~

Reiniciamos el servicio:

~~~
sudo systemctl restart squid
~~~


## Configurar nuestro ordenador para usar el proxy

Vamos a configurar nuestro ordenador para usar el proxy de dos maneras diferentes, desde el navegador o desde el proxy del sistema.

### Proxy navegador

Configuramos el navegador:

![1](/assets/img/posts/squid/1.png)

Y comprobamos que podemos navegar:

![2](/assets/img/posts/squid/2.png)

Y si comprobamos en el fichero de log del servidor proxy:

~~~
sudo less /var/log/squid/access.log

1614769538.485 116086 192.168.200.1 TCP_TUNNEL/200 3967 CONNECT dmpue.el-mundo.net:443 - HIER_DIRECT/40.118.29.72 -
1614769539.485 116019 192.168.200.1 TCP_TUNNEL/200 3544 CONNECT dmpue.el-mundo.net:443 - HIER_DIRECT/40.118.29.72 -
1614769548.541  73194 192.168.200.1 TCP_TUNNEL/200 49066 CONNECT dit.gonzalonazareno.org:443 - HIER_DIRECT/80.59.1.152 -
1614769548.541  73200 192.168.200.1 TCP_TUNNEL/200 49247 CONNECT dit.gonzalonazareno.org:443 - HIER_DIRECT/80.59.1.152 -
1614769548.541  76027 192.168.200.1 TCP_TUNNEL/200 165300 CONNECT dit.gonzalonazareno.org:443 - HIER_DIRECT/80.59.1.152 -
1614769548.745  73401 192.168.200.1 TCP_TUNNEL/200 100400 CONNECT dit.gonzalonazareno.org:443 - HIER_DIRECT/80.59.1.152 -
~~~

### Proxy sistema

Antes de nada, ponemos que el navegador use el proxy del sistema:

![3](/assets/img/posts/squid/3.png)

Y seguidamente, en las opciones del proxy del sistema, ponemos la `ip` del servidor:

![4](/assets/img/posts/squid/4.png)

Comprobamos que podemos navegar:

![5](/assets/img/posts/squid/5.png)

Y comprobamos el fichero log:

~~~
sudo less /var/log/squid/access.log

1614769898.759    196 192.168.200.1 TCP_MISS/503 4357 GET http://josedomingo.org/ - HIER_NONE/- text/html
1614769898.856      0 192.168.200.1 TCP_DENIED/403 4199 GET http://proxy:3128/squid-internal-static/icons/SN.png - HIER_NONE/- text/html
1614769898.862      0 192.168.200.1 TCP_MISS/503 4278 GET http://josedomingo.org/favicon.ico - HIER_NONE/- text/html
1614769904.599    158 192.168.200.1 TCP_MISS/301 646 GET http://www.josedomingo.org/ - HIER_DIRECT/37.187.119.60 text/html
1614769904.989    253 192.168.200.1 TCP_MISS/200 981 POST http://r3.o.lencr.org/ - HIER_DIRECT/212.230.153.40 application/ocsp-response
~~~

## Configurando squid para que se use desde el cliente interno

Desde el servidor, abrimos el fichero de configuración de squid:

~~~
sudo nano /etc/squid/squid.conf
~~~

Y añadimos otra `acl` y regla para poder navegar para la red interna:

~~~
acl interna src 10.0.0.0/24
http_access allow interna
~~~

Reiniciamos el servicio:

~~~
sudo systemctl restart squid
~~~

Y ahora desde el cliente, ponemos la nueva variable para poder configurar el proxy:

~~~
export http_proxy='http://10.0.0.10:3128'
~~~

### Comprobación cliente

Como únicamente tenemos salida por protocolo `HTTP`, lo que vamos a hacer será descargar el `index.html` de cualquier página para comprobar que está funcionando, para ello:

~~~
wget acabe10.github.io
--2021-03-03 11:25:22--  http://acabe10.github.io/
Connecting to 10.0.0.10:3128... connected.
Proxy request sent, awaiting response... 301 Moved Permanently
Location: https://acabe10.github.io/ [following]
--2021-03-03 11:25:22--  https://acabe10.github.io/
Resolving acabe10.github.io (acabe10.github.io)... 185.199.110.153, 185.199.108.153, 185.199.109.153, ...
Connecting to acabe10.github.io (acabe10.github.io)|185.199.110.153|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 10872 (11K) [text/html]
Saving to: ‘index.html.1’

index.html.1         100%[=====================>]  10.62K  --.-KB/s    in 0.001s  

2021-03-03 11:25:22 (14.0 MB/s) - ‘index.html.1’ saved [10872/10872]
~~~

Podemos comprobar igualmente el log:

~~~
sudo cat /var/log/squid/access.log

1614770694.644     56 10.0.0.11 TCP_MISS/301 642 GET http://google.es/ - HIER_DIRECT/142.250.184.163 text/html
1614770694.747    101 10.0.0.11 TCP_MISS/200 14671 GET http://www.google.es/ - HIER_DIRECT/216.58.201.131 text/html
1614770722.665    257 10.0.0.11 TCP_MISS/301 741 GET http://acabe10.github.io/ - HIER_DIRECT/185.199.111.153 text/html
~~~

## Lista negra por dominio

Para hacer una lista negra, primero vamos a crear el fichero con los dominios que no queramos que se puedan acceder:

~~~
sudo nano /etc/squid/blacklist
~~~

Y añadimos algo de contenido:

~~~
.tinaja.es
~~~

Ahora vamos a indicarle en el fichero de configuración que usaremos dicha lista negra, creando una nueva `ACL`, para ello:

~~~
sudo nano /etc/squid/squid.conf
~~~

Y añadimos lo siguiente:

~~~
acl blacklist dstdomain "/etc/squid/blacklist"
http_access deny blacklist
~~~

Reiniciamos el servicio:

~~~
sudo systemctl restart squid
~~~

### Comprobación en equipo

En nuestro equipo, comprobamos si podemos acceder:

![6](/assets/img/posts/squid/6.png)

## Lista blanca por dominio

Ahora configuraremos una lista blanca, para ello, vamos a crear un fichero con los dominios que queremos que sean los únicos accesibles:

~~~
sudo nano /etc/squid/whitelist

.acabe10.github.io
~~~

Editamos el fichero de configuración de squid:

~~~
sudo nano /etc/squid/squid.conf
~~~

Y añadimos lo siguiente:

~~~
acl whitelist dstdomain "/etc/squid/whitelist"
http_access deny !whitelist
~~~

Reiniciamos squid:

~~~
sudo systemctl restart squid
~~~

### Comprobación

Y desde nuestro equipo comprobamos que sólo podemos acceder a dicha página:

![7](/assets/img/posts/squid/7.png)

![8](/assets/img/posts/squid/8.png)