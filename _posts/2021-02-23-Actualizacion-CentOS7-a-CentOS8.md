---
layout: post
title: Actualización CentOS 7 a CentOS 8
tags: [all, OpenStack, Toboso, CentOS]
---
# Introducción

Buenas, en este post vamos a actualizar la instancia <code>quijote</code> que tenemos en nuestro escenario de OpenStack. La actualizaremos de CentOS 7 a CentOS 8.

## Comprobamos la versión de quijote

~~~
cat /etc/redhat-release

CentOS Linux release 7.9.2009 (Core)
~~~

## Empezamos con lo necesario para la actualización:

Todos los pasos que haremos a continuación son obviamente con root o sudo.

- Instalamos el repositorio de <code>EPEL</code>:

	~~~
	yum install epel-release -y
	~~~

- Instalamos las utilidades necesarias que nos harán falta:

	~~~
	yum install yum-utils -y
	~~~

- Instalamos el paquete para resolver los paquetes <code>RPM</code>:

	~~~
	yum install rpmconf -y
	~~~

- Y los resolvemos:

	~~~
	rpmconf -a
	~~~

- Limpiamos los paquetes innecesarios:

	~~~
	package-cleanup --leaves
	package-cleanup --orphans
	~~~

- Instalamos <code>dnf</code>, ya que en CentOS 8 no se usa <code>yum</code>:

	~~~
	yum install dnf -y
	~~~

- Eliminamos <code>yum</code>:

	~~~
	dnf -y remove yum yum-metadata-parser 
	rm -Rf /etc/yum
	~~~

- Instalamos el paquete de lanzamiento de CentOS 8, los cuáles los podemos encontrar en la siguiente dirección:

	http://mirror.centos.org/centos/8/BaseOS/x86_64/os/Packages/

	~~~
	dnf install http://mirror.centos.org/centos/8/BaseOS/x86_64/os/Packages/centos-repos-8.2-2.2004.0.1.el8.x86_64.rpm http://mirror.centos.org/centos/8/BaseOS/x86_64/os/Packages/centos-release-8.2-2.2004.0.1.el8.x86_64.rpm http://mirror.centos.org/centos/8/BaseOS/x86_64/os/Packages/centos-gpg-keys-8.2-2.2004.0.1.el8.noarch.rpm
	~~~

- Actualizamos el repositorio de <code>EPEL</code>:

	~~~
	dnf -y upgrade epel-release
	~~~

- Eliminamos los archivos temporales:

	~~~
	dnf makecache
	dnf clean all
	~~~

- Eliminamos el kérnel antiguo de CentOS 7:

	~~~
	grub2-mkconfig -o /boot/grub2/grub.cfg
	rpm -e `rpm -q kernel`
	~~~

- Eliminamos los paquetes conflictivos:

	~~~
	rpm -e --nodeps sysvinit-tools
	~~~

## Actualizamos

- Lanzamos la actualización al nuevo sistema:

	~~~
	dnf -y --releasever=8 --allowerasing --setopt=deltarpm=false distro-sync 
	~~~

- A nosotros nos ha dado un error con las dependencias de <code>Python 3</code>, el cuál hemos resuelto de la siguiente forma:

	~~~
	rpm -e --justdb python36-rpmconf-1.0.22-1.el7.noarch rpmconf-1.0.22-1.el7.noarch
	rpm -e --justdb --nodeps python3-setuptools-39.2.0-10.el7.noarch
	rpm -e --justdb --nodeps python3-pip-9.0.3-8.el7.noarch
	rpm -e --justdb --nodeps vim-minimal
	~~~

- Volvemos a ejecutar:

	~~~
	dnf -y --releasever=8 --allowerasing --setopt=deltarpm=false distro-sync 
	~~~

- Instalamos el nuevo kérnel para CentOS 8:

	~~~
	dnf -y install kernel-core
	~~~

- Comprobamos su instalación:

	~~~
	uname -r

	4.18.0-193.28.1.el8_2.x86_64
	~~~

- Instalamos el paquete mínimo de CentOS 8:

	~~~
	dnf -y groupupdate "Core" "Minimal Install"
	~~~

Y verificamos que se ha actualizado correctamente:

	~~~
	cat /etc/redhat-release

	CentOS Linux release 8.2.2004 (Core)
	~~~

Al reiniciar nos hemos dado cuenta de que nos tardaba bastante en hacerlo, para solucionarlo hemos deshabilitado el cloud-init, ya que estaba intentando resolver el direccionamiento mediante DHCP, y no tenemos DHCP en nuestra red, para deshabilitarlo:

~~~
touch /etc/cloud/cloud-init.disabled
~~~