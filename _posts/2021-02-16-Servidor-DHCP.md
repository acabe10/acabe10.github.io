---
layout: post
title: Servidor DHCP
tags: [DHCP, Servidor]
---
# Introducción

Buenas, vamos a explicar de forma resumida qué es y los estados de un servidor DHCP y posteriormente haremos ejercicios al respecto.

# ¿Qué es un servidor DHCP?

La función de un servidor DHCP es poder ofrecer a los hosts de su red direcciones IP y otros parámetros TCP/IP(puerta de enlace, dirección broadcast..etc)

Un servidor DHCP tiene diferentes estados:

## INIT

Un cliente DHCP enciende su máquina y envía un mensaje DHCPDISCOVER encapsulado en un paquete UDP para comprobar si existe algún servidor DHCP. Esté mensaje usa la dirección IP de broadcast 255.255.255.255.

## SELECTING

Una vez enviado el mensaje, el cliente entra en estado SELECTING, donde recibe mensajes DHCPOFFER de los servidores de la red. Si hay varios servidores, el cliente envía un mensaje DHCPREQUEST para elegir uno, el que contestará con un DHCPACK.

Si la red tiene broadcast, el cliente envía una petición ARP con la dirección IP sugerida, si esta está en uso, volverá a estado INIT para solicitar otra.

## BOUND

Cuando se acepta el DHPACK, el servidor pasa 3 valores al cliente de temporización:

- T1: temporizador de renovación de alquiler.
- T2: temporizador de reenganche.
- T3: duración de alquiler.

Este mensaje siempre ofrece el valor T3. Los valores T1 y T2 se configurán en el servidor o los da por defecto.

## RENEWING

Cuando el T1 expira, el cliente entra en estado RENEWING. En el cuál el cliente intentará negociar un nuevo alquiler para su dirección IP. Si el servidor DHCP que se la ofreció, no renueva, le enviará un mensaje DHCPNACK y el cliente entrará en INIT. Si el servidor le envía un mensaje DHCPACK, éste contendrá la duración del nuevo alquiler.

Después de esto entrará de nuevo en BOUND.

## REBINDING

Si el temporizador T2 expira mientras el cliente está en estado RENEWING, entrará en estado REBINDING.

### Extensión de la concesión

El cliente enviará un broadcast DHCPREQUEST para contactar con cualquier servidor DHCP para extender el alquiler porque se supone que el DHCP original estará apagado o no existe conexión. Si algún servidor DHCP responde con un DHCPACK, el cliente renueva su alquier(T3) y vuelve al estado BOUND. Si no hay servidor que responda antes de que expire el T3, el cliente vuelve a INIT.

### Expiración de la concesión

Al expirar el T3, el cliente debe devolver su dirección IP. Aunque el cliente también puede renunciar voluntariamente a una dirección IP, cuando lo hace, el cliente envia un mensaje DHCPRELEASE al servidor DHCP.

# Fichero escenario a crear

Vamos a crear un VagrantFile para definir el escenario. En el VagrantFile hemos añadido la instalación y configuración del servidor y la configuración del cliente:

~~~
# -*- mode: ruby -*-
	# vi: set ft=ruby :

	Vagrant.configure("2") do |config|

		config.vm.define :server do |server|
			server.vm.box = "debian/buster64"
			server.vm.hostname = "server"
			server.vm.network "public_network",:bridge=>"wlp5s0"
			server.vm.network "private_network", ip: "192.168.100.1",
				virtualbox__intnet: "intranet"
			server.vm.provision "shell",run: "always", inline: <<-script
				# Actualizamos máquina y ponemos la instalación de grub por defecto
				sudo apt-get update
				sudo echo "SET grub-pc/install_devices /dev/sda1" | sudo debconf-communicate
				sudo apt-get upgrade -y

				# Instalaciones necesarias
				sudo apt-get -y install isc-dhcp-server iptables

				# Configuración DHCP

				sudo echo 'INTERFACESv4="eth2"' > /etc/default/isc-dhcp-server
				sudo echo 'INTERFACESv6=""' >> /etc/default/isc-dhcp-server

				sudo echo "default-lease-time 600;" > /etc/dhcp/dhcpd.conf
				sudo echo "max-lease-time 43200;" >> /etc/dhcp/dhcpd.conf
				sudo echo "ddns-update-style none;" >> /etc/dhcp/dhcpd.conf
				sudo echo "subnet 192.168.100.0 netmask 255.255.255.0" >> /etc/dhcp/dhcpd.conf
	  			sudo echo "		{" >> /etc/dhcp/dhcpd.conf
	  			sudo echo "		range 192.168.100.100 192.168.100.120;" >> /etc/dhcp/dhcpd.conf
	  			sudo echo "		option routers 192.168.100.1;" >> /etc/dhcp/dhcpd.conf
	  			sudo echo "		option domain-name-servers 8.8.8.8, 4.4.4.4;" >> /etc/dhcp/dhcpd.conf
	  			sudo echo "		}" >> /etc/dhcp/dhcpd.conf
				
	  			sudo systemctl restart isc-dhcp-server.service
				
	  			# Activar bit de forwarding
	  			sudo sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/g' /etc/sysctl.conf
				sudo sysctl -p /etc/sysctl.conf

				# Acceso a exterior
				sudo iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o eth1 -j MASQUERADE
				
				# Rutas por defecto
				sudo ip r del default via 10.0.2.2
				sudo ip r add default via 192.168.1.1

				script
		end
	  
		config.vm.define :client1 do |client1|
			client1.vm.box = "debian/buster64"
			client1.vm.hostname = "client1"
			client1.vm.network "private_network", type: "dhcp",
				virtualbox__intnet: "intranet"
			client1.vm.provision "shell", inline: <<-script_2
				# Nos liberamos y renovamos ip
				sudo dhclient -r

				# Rutas por defecto
				sudo ip r del default via 10.0.2.2
				sudo ip r add default via 192.168.100.1
				script_2
		end
	end
~~~

## Comprobaciones y ficheros

### Lista de concesiones

~~~
vagrant@server:~$ cat /var/lib/dhcp/dhcpd.leases
	# The format of this file is documented in the dhcpd.leases(5) manual page.
	# This lease file was written by isc-dhcp-4.4.1

	# authoring-byte-order entry is generated, DO NOT DELETE
	authoring-byte-order little-endian;

	server-duid "\000\001\000\001'\031\2236\010\000'\335TA";

	lease 192.168.100.100 {
	  starts 3 2020/10/14 10:45:51;
	  ends 3 2020/10/14 10:55:51;
	  cltt 3 2020/10/14 10:45:51;
	  binding state active;
	  next binding state free;
	  rewind binding state free;
	  hardware ethernet 08:00:27:22:74:d4;
	  uid "\377'\"t\324\000\001\000\001'\031\223]\010\000'\"t\324";
	  client-hostname "client1";
	}
~~~

### Comando "ip a" client1

~~~
vagrant@client1:~$ ip a
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	    inet 127.0.0.1/8 scope host lo
	       valid_lft forever preferred_lft forever
	    inet6 ::1/128 scope host 
	       valid_lft forever preferred_lft forever
	2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
	    link/ether 08:00:27:8d:c0:4d brd ff:ff:ff:ff:ff:ff
	    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
	       valid_lft 85531sec preferred_lft 85531sec
	    inet6 fe80::a00:27ff:fe8d:c04d/64 scope link 
	       valid_lft forever preferred_lft forever
	3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
	    link/ether 08:00:27:22:74:d4 brd ff:ff:ff:ff:ff:ff
	    inet 192.168.100.100/24 brd 192.168.100.255 scope global dynamic eth1
	       valid_lft 504sec preferred_lft 504sec
	    inet6 fe80::a00:27ff:fe22:74d4/64 scope link 
	       valid_lft forever preferred_lft forever
~~~

### Ping a www.google.es desde client1

![img1](/assets/img/posts/dhcp/img1.png)

### tcpdump en servidor

Capturamos con el servidor:

~~~
sudo tcpdump -i eth2 -v
~~~

Y con el cliente hacemos:

~~~
sudo dhclient -r
sudo dhclient
~~~

Y el servidor nos capturará los paquetes discover:

![discover](/assets/img/posts/dhcp/discover.png)

offer:

![offer](/assets/img/posts/dhcp/offer.png)

request:

![request](/assets/img/posts/dhcp/request.png)

Y ack:

![ack](/assets/img/posts/dhcp/ack.png)

### Rutas en client1

~~~
vagrant@client1:~$ ip r
default via 192.168.100.1 dev eth1 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 
192.168.100.0/24 dev eth1 proto kernel scope link src 192.168.100.100
~~~

### Rutas en server

~~~
default via 192.168.1.1 dev eth1 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 
192.168.1.0/24 dev eth1 proto kernel scope link src 192.168.1.193 
192.168.100.0/24 dev eth2 proto kernel scope link src 192.168.100.1
~~~

# Creación de reservas

Para hacer la reserva debemos de editar el fichero */etc/dhcp/dhcpd.conf* añadiendo lo siguiente:

~~~
host client1 {
 hardware ethernet 08:00:27:71:2b:8f;
 fixed-address 192.168.100.150;
}
~~~

## Comprobación

![reserva](/assets/img/posts/dhcp/reserva.png)

# Modificamos escenario

Modifica el escenario Vagrant para añadir una nueva red local y un nuevo nodo:

Servidor: En el servidor hay que crear una nueva interfaz
nodo_lan2: Un cliente conectado a la segunda red local.
Configura el servidor dhcp en el ordenador “servidor” para que de servicio a los ordenadores de la nueva red local, teniendo en cuenta que el tiempo de concesión sea 24 horas y que la red local tiene el direccionamiento 192.168.200.0/24.

Hemos definido el nuevo _VagrantFile_ para que cree directamente la segunda red local 192.168.200.0/24, que cree también el segundo nodo y que haga la regla de iptables que permita la salida a la segunda red local:

~~~
# -*- mode: ruby -*-
	# vi: set ft=ruby :

	Vagrant.configure("2") do |config|

		config.vm.define :server do |server|
			server.vm.box = "debian/buster64"
			server.vm.hostname = "server"
			server.vm.network "public_network",:bridge=>"wlp5s0"
			server.vm.network "private_network", ip: "192.168.100.1",
				virtualbox__intnet: "intranet"
			server.vm.network "private_network", ip: "192.168.200.1",
				virtualbox__intnet: "intranet_2"
			server.vm.provision "shell",run: "always", inline: <<-script
				# Actualizamos máquina y ponemos la instalación de grub por defecto
				sudo apt-get update
				sudo echo "SET grub-pc/install_devices /dev/sda1" | sudo debconf-communicate
				sudo apt-get upgrade -y

				# Instalaciones necesarias
				sudo apt-get -y install isc-dhcp-server iptables

				# Configuración DHCP lan1

				sudo echo "ddns-update-style none;" > /etc/dhcp/dhcpd.conf
				sudo echo "subnet 192.168.100.0 netmask 255.255.255.0" >> /etc/dhcp/dhcpd.conf
	  			sudo echo "		{" >> /etc/dhcp/dhcpd.conf
				sudo echo "		default-lease-time 600;" >> /etc/dhcp/dhcpd.conf
				sudo echo "		max-lease-time 43600;" >> /etc/dhcp/dhcpd.conf
	  			sudo echo "		range 192.168.100.100 192.168.100.120;" >> /etc/dhcp/dhcpd.conf
	  			sudo echo "		option routers 192.168.100.1;" >> /etc/dhcp/dhcpd.conf
	  			sudo echo "		option domain-name-servers 8.8.8.8, 4.4.4.4;" >> /etc/dhcp/dhcpd.conf
	  			sudo echo "		}" >> /etc/dhcp/dhcpd.conf

	  			# Configuración DHCP lan2

				sudo echo 'INTERFACESv4="eth2 eth3"' > /etc/default/isc-dhcp-server
				sudo echo 'INTERFACESv6=""' >> /etc/default/isc-dhcp-server

				sudo echo "subnet 192.168.200.0 netmask 255.255.255.0" >> /etc/dhcp/dhcpd.conf
	  			sudo echo "		{" >> /etc/dhcp/dhcpd.conf
	  			sudo echo "		default-lease-time 600;" >> /etc/dhcp/dhcpd.conf
				sudo echo "		max-lease-time 21600;" >> /etc/dhcp/dhcpd.conf
	  			sudo echo "		range 192.168.200.100 192.168.200.120;" >> /etc/dhcp/dhcpd.conf
	  			sudo echo "		option routers 192.168.200.1;" >> /etc/dhcp/dhcpd.conf
	  			sudo echo "		option domain-name-servers 8.8.8.8, 4.4.4.4;" >> /etc/dhcp/dhcpd.conf
	  			sudo echo "		}" >> /etc/dhcp/dhcpd.conf
				
	  			sudo systemctl restart isc-dhcp-server.service
				
	  			# Activar bit de forwarding
	  			sudo sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/g' /etc/sysctl.conf
				sudo sysctl -p /etc/sysctl.conf

				# Acceso a exterior lan1
				sudo iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o eth1 -j MASQUERADE

				# Acceso a exterior lan2
				sudo iptables -t nat -A POSTROUTING -s 192.168.200.0/24 -o eth1 -j MASQUERADE
				
				# Rutas por defecto
				sudo ip r del default via 10.0.2.2
				sudo ip r add default via 192.168.1.1

				script
		end
	  
		config.vm.define :client1 do |client1|
			client1.vm.box = "debian/buster64"
			client1.vm.hostname = "client1"
			client1.vm.network "private_network", type: "dhcp",
				virtualbox__intnet: "intranet"
			client1.vm.provision "shell", inline: <<-script_2
				# Rutas por defecto
				sudo ip r del default via 10.0.2.2
				sudo ip r add default via 192.168.100.1
				script_2
		end

		config.vm.define :client2 do |client2|
			client2.vm.box = "debian/buster64"
			client2.vm.hostname = "client2"
			client2.vm.network "private_network", type: "dhcp",
				virtualbox__intnet: "intranet_2"
			client2.vm.provision "shell", inline: <<-script_3
				# Rutas por defecto
				sudo ip r del default via 10.0.2.2
				sudo ip r add default via 192.168.200.1
				script_3
		end
	end
~~~

## Comprobaciones

### Fichero dhcpd.conf

~~~
subnet 192.168.200.0 netmask 255.255.255.0
		{
		default-lease-time 600;
		max-lease-time 21600;
		range 192.168.200.100 192.168.200.120;
		option routers 192.168.200.1;
		option domain-name-servers 8.8.8.8, 4.4.4.4;
		}
~~~

### Ips server, client1 y client2

![ips](/assets/img/posts/dhcp/ips.png)

### Fichero de concesiones

![concesiones](/assets/img/posts/dhcp/concesiones.png)

### Ip a google desde client2

![img17](/assets/img/posts/dhcp/ipgoo.png)