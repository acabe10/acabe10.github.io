---
layout: post
title: Introducción a Vagrant
tags: [all, Vagrant, instalación]
---
## Introducción

Buenas, en este post vamos a hacer varios ejercicios para aprender a usar Vagrant, una aplicacion libre desarrollada en ruby que nos permite crear y personalizar entornos de desarrollo livianos, reproducibles y portables. Vagrant nos permite automatizar la creación y gestión de máquinas virtuales.

Las máquinas virtuales creadas por vagrant se pueden ejecutar con distintos gestores de máquinas virtuales (oficialmente VirtualBox, VMWare e Hyper-V), en nuestro ejemplo vamos a usar máquinas virtuales en VirtualBox.

## Instalación de Vagrant

Instalamos VirtualBox y Vagrant (en Debian Buster lo podemos instalar de repositorios, si queremos la última versión lo podemos descargar desde la [página oficial](https://www.vagrantup.com/downloads)):

~~~
# apt install virtualbox-6.0 vagrant
~~~

## Añadir box de Vagrant

Los boxes de Vagrant son unas imágenes muy optimizadas, son sistemas operativos preinstalados, a partir de los cuales se crean las máquinas virtuales, que una vez creadas se pueden provisionar con el software que se necesite en cada caso.

Tenemos una [lista de boxes](https://app.vagrantup.com/boxes/search) los cuáles podemos descargar.

Para descargar un box lo hacemos con usuario sin privilegio:

~~~
$ vagrant box add debian/buster64
~~~

Ya que cada usuario tiene su propia lista de boxes:

~~~
$ vagrant box list
~~~

## Creación de una máquina virtual

1. Para ello, vamos a crear un directorio y dentro vamos a crear el fichero _VagrantFile_, que es dónde van a estar todos los parámetros para la creación de la máquina. Podemos usar uno vacío con la instrucción:

	~~~
	$ vagrant init
	~~~

2. Modificamos el fichero _VagrantFile_ y lo dejamos de la siguiente manera:

	~~~
	# -*- mode: ruby -*-
	# vi: set ft=ruby :
	Vagrant.configure("2") do |config|
		config.vm.box = "debian/buster64"
		config.vm.hostname = "mimaquina"
		config.vm.network :public_network,:bridge=>"enp3s0"
	end
	~~~

	Como vemos, se creará una máquina simple, con una red en modo puente por _enp3s0_.

3. Levantamos la máquina:

	~~~
	$ vagrant up
	~~~

4. Para acceder a la máquina:

	~~~
	$ vagrant ssh default
	~~~

	o si sólo tenemos una máquina:

	~~~
	$ vagrant ssh
	~~~

5. Suspender, apagar o destruir:

	~~~
	$ vagrant suspend
	$ vagrant halt
	$ vagrant destroy
	~~~

## Creación de varias máquinas virtuales

En esta ocasión vamos a crear otro directorio y dentro un fichero _VagrantFile_ con el siguiente contenido:

~~~
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|

	config.vm.define :nodo1 do |nodo1|
		nodo1.vm.box = "debian/buster64"
		nodo1.vm.hostname = "nodo1"
		nodo1.vm.network :private_network, ip: "10.1.1.101"
	end

	config.vm.define :nodo2 do |nodo2|
		nodo2.vm.box = "debian/buster64"
		nodo2.vm.hostname = "nodo2"
		nodo2.vm.network :public_network,:bridge=>"enp3s0"
		nodo2.vm.network :private_network, ip: "10.1.1.102"
	end
end
~~~

El anterior escenario nos creará dos máquinas virtuales(nodo1 y nodo2), nodo1 tendrá una interfaz privada con la ip 10.1.1.101 y nodo2 tendrá dos interfaces, una privada con la ip 10.1.1.102 y otra pública igual que la que hicimos en el ejercicio anterior.

## Añadir disco duro adicional y modificar RAM

Por último vamos a crear un nuevo _VagrantFile_ en un nuevo directorio con el siguiente contenido:

~~~
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

	config.vm.define :nodo1 do |nodo1|
		nodo1.vm.box = "debian/buster64"
		nodo1.vm.hostname = "nodo1"
		nodo1.vm.network :private_network, ip: "10.1.1.101"
		nodo1.vm.provider :virtualbox do |v|
			v.customize ["modifyvm", :id, "--memory", 1024]
		end
	end

	disco = '.vagrant/midisco.vdi'
	config.vm.define :nodo2 do |nodo2|
		nodo2.vm.box = "debian/buster64"
		nodo2.vm.hostname = "nodo2"
		nodo2.vm.network :public_network,:bridge=>"eth0"
		nodo2.vm.network :private_network, ip: "10.1.1.102"
		nodo2.vm.provider :virtualbox do |v|
			v.customize ["createhd", "--filename", disco, "--size", 1024]
			v.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", 1, "--device", 0, "--type", "hdd", "--medium", disco]
		end
    end
end
~~~

Como podemos ver al _nodo1_ le hemos modificado el tamaño de la memoria RAM y en el _nodo2_ hemos añadido un disco duro de 1GB. Para que estos cambios tengan efecto:

~~~
$ vagrant reload
~~~

Tenemos muchos más parámetros de configuración para _Vagrant_, puedes encontrarlos en la [documentación oficial](https://www.vagrantup.com/docs).