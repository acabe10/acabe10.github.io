---
layout: post
title: Instalación Oracle Enterprise Manager 12c sobre Debian
tags: [all, Oracle, Debian, BBDD]
---
# Introducción

Para la instalación de Oracle Enterprise Manager vamos a instalar Oracle Database 12c sobre Debian 8, el cuál trae Enterprise Manager incluido.

## Máquina en virtualbox

Tenemos una máquina en virtualbox con las siguientes características:

* 30GB de disco duro.
* 2GB de RAM

## Preinstalación

Para poder instalar Oracle database sobre Debian, tenemos que hacer una serie de requisitos como usuario root:

Añadimos usuarios y grupos necesarios:

~~~
addgroup --system oinstall

addgroup --system dba

adduser --system --ingroup oinstall -shell /bin/bash oracle

adduser oracle dba

passwd oracle
~~~

Creamos el archivo para las configuraciones de oracle:

~~~
fs.file-max = 65536

fs.aio-max-nr = 1048576

kernel.sem = 250 32000 100 128

kernel.shmmax = 2147483648

kernel.shmall = 2097152

kernel.shmmni = 4096

net.ipv4.ip_local_port_range = 104 65000

vm.hugetlb_shm_group = 128

vm.nr_hugepages = 64
~~~

Para obtener el `kernel.shmmax` y `kernel.shmall` ejecutamos el siguiente script como root:

~~~
#!/bin/bash
page_size=$(getconf PAGE_SIZE)
phys_pages=$(getconf _PHYS_PAGES)
shmall=$(( $phys_pages / 2 ))
shmmax=$(( $shmall * $page_size ))
echo kernel.shmmax = $shmmax
echo kernel.shmall = $shmall
exit 0
~~~

Cargamos la configuración anterior:

~~~
sysctl -p /etc/sysctl.d/local-oracle.conf
~~~

Añadimos los limites de seguridad de oracle al fichero:

~~~
nano /etc/security/limits.conf
~~~

Le añadimos el siguiente contenido:

~~~
oracle          soft    nproc           2047
oracle          hard    nproc           16384
oracle          soft    nofile          1024
oracle          hard    nofile          65536
oracle          soft    memlock         204800
oracle          hard    memlock         204800
~~~

Creamos los siguientes enlaces simbólicos:

~~~
ln -s /usr/bin/awk /bin/awk

ln -s /usr/bin/basename /bin/basename

ln -s /usr/bin/rpm /bin/rpm

ln -s /usr/lib/x86_64-linux-gnu /usr/lib64
~~~

Los directorios necesarios:

~~~
mkdir -p /opt/oracle/product/12.1.0.2
mkdir -p /opt/oraInventory
chown -R oracle:dba /opt/ora*
~~~

Paquetes necesarios:

~~~
apt install build-essential binutils libcap-dev gcc g++ libc6-dev ksh libaio-dev make libxi-dev libxtst-dev libxau-dev libxcb1-dev sysstat rpm xauth unzip
~~~

## Descarga de Oracle

Desde la página oficial de Oracle, nos registramos y descargamos la versión 12c para Linux. La descomprimimos en el directorio personal del usuario oracle:

~~~
unzip linuxamd64_12102_database_1of2.zip

unzip linuxamd64_12102_database_2of2.zip
~~~

Le damos permisos:

~~~
chown -R oracle:oinstall database
~~~

Accedemos al usuario oracle y declaramos las siguientes variables:

~~~
export ORACLE_HOSTNAME=localhost

export ORACLE_OWNER=oracle

export ORACLE_BASE=/opt/oracle

export ORACLE_HOME=/opt/oracle/product/12.1.0.2/base1

export ORACLE_UNQNAME=base1

export ORACLE_SID=base1

export PATH=$PATH:$ORACLE_HOME/bin

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib/x86_64-linux-gnu:/bin/lib:/lib/x86_64-linux-gnu/:/usr/lib64
~~~

## Instalación de Oracle

Ejecutamos el instalador desde local:

~~~
/home/oracle/database/runInstaller -IgnoreSysPreReqs
~~~

O desde ssh:

~~~
ssh -XC oracle@192.168.1.247 database/runInstaller -IgnoreSysPreReqs
~~~

Vamos a explicar de manera rápida los pasos para su instalación:

* No seleccionamos correo
* Crear y configurar base de datos
* Servidor
* Instancia única
* Instalación avanzada
* Idiomas por defecto
* Edición enterprise
* Ubicación por defecto: `/opt/oracle`
* Inventario por defecto `/opt/oracle/oraInventory`
* Configuración por defecto
* El ID nos debe de aparecer base1 si hemos introducido las variables correctamente
* Almacenamiento base de datos por defecto: `/opt/oracle/product/12.1.0.2`
* Gestión por defecto
* Recuperación por defecto
* Establecemos una contraseña
* Opción por defecto
* Instalamos
* Ejecutamos los scripts requeridos
* Finalizado

Para poder activar la base de datos, tendremos que iniciarla, para ello:

~~~
source /usr/local/bin/oraenv
ORACLE_SID = [base1] ?
~~~

Pulsamos intro.

Arrancamos la base:

~~~
dbstart $ORACLE_HOME
~~~

Entramos como administradores:

~~~
sqlplus SYS AS SYSDBA
~~~

Y ejecutamos:

~~~
> startup
> alter session set "_ORACLE_SCRIPT"=true;
> create user cabezas identified by cabezas;
> grant connect to cabezas;
> grant resource to cabezas;
> grant em_express_all to cabezas;
~~~

Ya podremos acceder a través de nuestro usuario cabezas por la terminal.

## Acceso mediante Enterprise Manager

Para poder acceder mediante Enterprise Manager, accedemos a la base de datos como:

~~~
sqlplus SYS AS SYSDBA
~~~

Y comprobamos el puerto que tenemos que definir:

~~~
> select dbms_xdb_config.getHTTPsport() from dual;

DBMS_XDB_CONFIG.GETHTTPSPORT()

------------------------------

 5500
~~~

Ejecutamos:

~~~
> exec dbms_xdb_config.setHTTPsport(5500);

PL/SQL procedure successfully completed.

> commit;

Commit complete.

>exit
~~~

Ahora nos vamos al direcorio(si no existe `bin` lo creamos manualmente)

~~~
/opt/oracle/product/12.1.0.2/dbhome_1/bin
~~~

Y ejecutamos:

~~~
lsnrctl start

LSNRCTL for Linux: Version 12.1.0.2.0 - Production on 15-DEC-2020 13:32:52

Copyright (c) 1991, 2014, Oracle.  All rights reserved.

TNS-01106: Listener using listener name LISTENER has already been started
~~~

## Comprobación

Desde otro cliente diferente accedemos mediante el navegador:

![12](/assets/img/posts/servidores-bd/12.png)

Introducimos la contraseña y podemos acceder al panel de control de `Enterprise Manager`:

![13](/assets/img/posts/servidores-bd/13.png)