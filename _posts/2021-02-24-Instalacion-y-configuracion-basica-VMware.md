---
layout: post
title: Instalación y configuración básica servidor VMware ESXi
tags: [all, VMware ESXi, Servidor]
---
# Introducción

Buenas, en este post vamos a explicar la configuración que hemos realizado para crear una máquina con acceso a internet en un servidor VMware ESXi previamente instalado.

Para instalarlo hemos usado un servidor físico que nos han cedido nuestros profesores. La instalación ha sido simplemente preparar un pendrive con `VMware ESXi` montado e instalarlo.

## Problemas al instalar VMware 5.5

En la instalación de VMware 5.5 hemos tenido un problema el cuál no cargaba correctamente el módulo IPMI, por ello hemos tenido que deshabilitarlo para que pudiera continuar con la instalación.

Para deshabilitarlo hemos entrado en la consola antes de que comienze la instalación pulsando `SHIFT+ o`, y seguidamente hemos puesto lo siguiente:

~~~
> noipmiEnabled
~~~

Hemos presionado ENTER y ha transcurrido la instalación con éxito.

## Acceso a VMware

Después de instalar VMware 5.5 en el servidor, ponerle nuestra puerta de enlace `172.22.0.1` y la IP `172.22.220.57`, hemos accedido mediante el navegador y hemos descargado desde el enlace el cliente de VMware:

![1](/assets/img/posts/vmware/1.png)

Una vez instalado, ponemos la ip de nuestra máquina y el usuario root:

![2](/assets/img/posts/vmware/2.png)

Nos avisa de que el certificado no es seguro, pero obviamente es nuestra máquina:

![3](/assets/img/posts/vmware/3.png)

Una vez dentro, tenemos el siguiente panel de control:

![4](/assets/img/posts/vmware/4.png)

El primer paso que tenemos que realizar es crear un espacio de almacenamiento para nuestras máquinas virtuales en nuestro servidor(storage), para ello nos vamos a la pestaña de `Configuration` > `Storage` > `Add Storage`:

![5](/assets/img/posts/vmware/5.png)

Seleccionamos Disk:

![6](/assets/img/posts/vmware/6.png)

Elegimos el disco que queramos:

![7](/assets/img/posts/vmware/7.png)

Dejamos la opcion por defecto:

![8](/assets/img/posts/vmware/8.png)

Dejamos usar espacio libre:

![9](/assets/img/posts/vmware/9.png)

Le damos un nombre al storage:

![10](/assets/img/posts/vmware/10.png)

El espacio que le queremos asignar:

![11](/assets/img/posts/vmware/11.png)

Confirmamos todos los datos:

![12](/assets/img/posts/vmware/12.png)

Una vez creado, nos aparecerá en la lista de storage:

![13](/assets/img/posts/vmware/13.png)

Creamos una nueva máquina:

![14](/assets/img/posts/vmware/14.png)

Opciones personalizadas:

![15](/assets/img/posts/vmware/15.png)

Le ponemos un nombre a la máquina:

![16](/assets/img/posts/vmware/16.png)

Elegimos el storage que usará:

![17](/assets/img/posts/vmware/17.png)

La versión de la máquina virtual:

![18](/assets/img/posts/vmware/18.png)

El sistema operativo que usaremos, nosotros hemos elegido `Others/Debian 64 bit`:

![19](/assets/img/posts/vmware/19.png)

Los cores que tendrá:

![20](/assets/img/posts/vmware/20.png)

Su memoria RAM:

![21](/assets/img/posts/vmware/21.png)

La tarjeta de red que usará:

![22](/assets/img/posts/vmware/22.png)

El controlador SCSI lo dejamos por defecto:

![23](/assets/img/posts/vmware/23.png)

El disco que tendrá la máquina:

![24](/assets/img/posts/vmware/24.png)

El tamaño y el aprovisionamiento, lo ponemos `Thick Provision Eager Zeroed`:

![25](/assets/img/posts/vmware/25.png)

Dejamos por defecto:

![26](/assets/img/posts/vmware/26.png)

Confirmamos las opciones y finalizamos:

![27](/assets/img/posts/vmware/27.png)

Encendemos la máquina:

![28](/assets/img/posts/vmware/28.png)

Pulsamos para añadir la ISO que instalaremos e instalamos el sistema operativo:

![29](/assets/img/posts/vmware/29.png)

Una vez instalado, comprobamos que funciona todo correctamente:

![30](/assets/img/posts/vmware/30.png)