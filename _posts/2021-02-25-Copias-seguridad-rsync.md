---
layout: post
title: Copias de seguridad con rsync
tags: [all, OpenStack, Toboso, rsync, copias de seguridad]
---
# Introducción

Buenas, en este post vamos a explicar el procedimiento que hemos realizado para tener un sistema de copias de seguridad en nuestro escenario de OpenStack.

Hemos elegido usar `rsync` y `ssh` en un script para hacer la transferencia segura. Hemos tenido que hacer una serie de requisitos para poder ejecutar el script correctamente. 

## Requisitos

### Instalación rsync

Hemos instalado `rsync` en todas las máquinas que vamos a hacer la copia de seguridad.

### Claves públicas

Hemos tenido que generar una clave pública para el usuario `root` en `freston` para que pueda entrar sin problemas en todas las máquinas.

### Clave GPG

Hemos generado una clave gpg para poder encriptar la carpeta `root` de cada máquina, para ello:

~~~
gpg --full-generate-key
gpg (GnuPG) 2.2.12; Copyright (C) 2018 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

gpg: directory '/root/.gnupg' created
gpg: keybox '/root/.gnupg/pubring.kbx' created
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 
Requested keysize is 3072 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Cabezas
Email address: alejandrocabezab@gmail.com
Comment: 

┌──────────────────────────────────────────────────────┐
                           │ Please enter the passphrase to                       │
                           │ protect your new key                                 │
                           │                                                      │
                           │ Passphrase: ****************________________________ │
                           │                                                      │
                           │       <OK>                              <Cancel>     │
                           └──────────────────────────────────────────────────────┘

You selected this USER-ID:
    "Cabezas <alejandrocabezab@gmail.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: /root/.gnupg/trustdb.gpg: trustdb created
gpg: key 8C3E0E8B2235C0C6 marked as ultimately trusted
gpg: directory '/root/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/root/.gnupg/openpgp-revocs.d/28172DEB5D575F711342CAB08C3E0E8B2235C0C6.rev'
public and secret key created and signed.

pub   rsa3072 2021-01-24 [SC]
      28172DEB5D575F711342CAB08C3E0E8B2235C0C6
uid                      Cabezas <alejandrocabezab@gmail.com>
sub   rsa3072 2021-01-24 [E]
~~~

### Disco adicional

Hemos instalado un disco adicional de 4GB en freston:

~~~
lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    254:0    0  10G  0 disk 
└─vda1 254:1    0  10G  0 part /
vdb    254:16   0   4G  0 disk
~~~

Hemos elegido hacer una copia incremental cada día de la semana, ya que está cogerá los ficheros modificados desde la última copia, ya sea completa o incremental. Le hemos dado un formato al disco en `btrfs`:

~~~
mkfs.btrfs -f /dev/vdb
btrfs-progs v4.20.1 
See http://btrfs.wiki.kernel.org for more information.

Label:              (null)
UUID:               792c8b53-b41b-49fd-b0aa-d0e1305d58af
Node size:          16384
Sector size:        4096
Filesystem size:    5.00GiB
Block group profiles:
  Data:             single            8.00MiB
  Metadata:         DUP             256.00MiB
  System:           DUP               8.00MiB
SSD detected:       no
Incompat features:  extref, skinny-metadata
Number of devices:  1
Devices:
   ID        SIZE  PATH
    1     5.00GiB  /dev/vdb
~~~

Y lo hemos montado automáticamente con la compresión `zlib` de `btrfs`, así nos podrá guarda más copias:

~~~
UUID=792c8b53-b41b-49fd-b0aa-d0e1305d58af       /root/backups    btrfs   compress=zlib   0       1
~~~

### Directorios a crear

Hemos creado unos directorios en freston que han quedado de la siguiente forma:

~~~
tree
├── backups
│   ├── dulcinea
│   │   ├── completas
│   │   │   └── latest
│   │   └── incrementales
│   │       └── latest
│   ├── freston
│   │   ├── completas
│   │   │   └── latest
│   │   └── incrementales
│   │       └── latest
│   ├── quijote
│   │   ├── completas
│   │   │   └── latest
│   │   └── incrementales
│   │       └── latest
│   └── sancho
│       ├── completas
│       │   └── latest
│       └── incrementales
│           └── latest
~~~

### Tarea crontab

Hemos añadido la siguiente tarea a crontab para que se ejecute todos los días a las 23:15:

~~~
crontab -e
~~~

Y añadimos lo siguiente al final del fichero:

~~~
15 23 * * * /root/backups/s_backup.sh
~~~

## Script

Y por último, muestro el script que se ejecutará diariamente:

~~~
#!/bin/bash

# Declaramos variable "fecha" para ver el día de hoy, el día de la copia del anterior domingo y el día anterior para ver las copias incrementales
fecha_hoy=$(date +%d-%m-%Y)
fecha_semana=$(date +%d-%m-%Y --date='- 7day')
fecha_dia_menos=$(date +%d-%m-%Y --date='- 1day')

# Declaramos variable "exclude" para excluir directorios a copiar
exclude={"/bin","/usr","/initrd.img.old","/lib64","/media","/proc","/sbin","/tmp","/vmlinuz*","/boot","/lib","/libx32","/mnt","/dev","/initrd.img","/lib32","/lost+found","/opt","/run","/sys"}

# Indicamos una condición si el día es domingo(fin semana)
if [[ $(date +%u) -eq 7 ]]; then

## DULCINEA ##
	## Copia semana anterior ##
	# Si existe dicha copia..

	if [ -d /root/backups/dulcinea/completas/completa-$fecha_semana ]; then
		# Comprimimos la carpeta root para encriptarla y eliminamos la anterior
		cd /root/backups/dulcinea/completas/completa-$fecha_semana && tar cfz root.tar.gz root && gpg --encrypt --recipient Cabezas root.tar.gz && rm -rf root && rm root.tar.gz
		# Comprimimos la copia de seguridad y eliminamos la carpeta
		cd .. && tar cfz completa-$fecha_semana.tar.gz completa-$fecha_semana && rm -rf completa-$fecha_semana
	fi

	# Hacemos copia completa a dulcinea y la envíamos a freston
	ssh root@dulcinea "rsync -tazhe ssh --exclude=$exclude / root@freston:backups/dulcinea/completas/completa-$fecha_hoy"
	
	# Mandamos correo si la copia ha salido correcta/incorrecta
	fail_com_dul=$?

	if [ $fail_com_dul -eq 0 ]; then
    	mail -s "Copia completa dulcinea correcta" alejandrocabezab@gmail.com <<< ""
	else
		mail -s "Copia completa dulcinea incorrecta" alejandrocabezab@gmail.com <<< ""
	fi

	# Guardamos nombre última copia
	echo completa-$fecha_hoy > /root/backups/dulcinea/completas/latest


	# Ponemos última copia incremental vacía
	echo "" > /root/backups/dulcinea/incrementales/latest

## SANCHO ##
	## Copia semana anterior ##
	# Si existe dicha copia..

	if [ -d /root/backups/sancho/completas/completa-$fecha_semana ]; then
		# Comprimimos la carpeta root para encriptarla y eliminamos la anterior
		cd /root/backups/sancho/completas/completa-$fecha_semana && tar cfz root.tar.gz root && gpg --encrypt --recipient Cabezas root.tar.gz && rm -rf root && rm root.tar.gz
		# Comprimimos la copia de seguridad y eliminamos la carpeta
		cd .. && tar cfz completa-$fecha_semana.tar.gz completa-$fecha_semana && rm -rf completa-$fecha_semana
	fi

	# Hacemos copia completa a sancho y la envíamos a freston
	ssh root@sancho "rsync -tazhe ssh --exclude=$exclude / root@freston:backups/sancho/completas/completa-$fecha_hoy"
	
	# Mandamos correo si la copia ha salido correcta/incorrecta
	fail_com_san=$?

	if [ $fail_com_dul -eq 0 ]; then
    	mail -s "Copia completa sancho correcta" alejandrocabezab@gmail.com <<< ""
	else
		mail -s "Copia completa sancho incorrecta" alejandrocabezab@gmail.com <<< ""
	fi

	# Guardamos nombre última copia
	echo completa-$fecha_hoy > /root/backups/sancho/completas/latest


	# Ponemos última copia incremental vacía
	echo "" > /root/backups/sancho/incrementales/latest

## QUIJOTE ##
	## Copia semana anterior ##
	# Si existe dicha copia..

	if [ -d /root/backups/quijote/completas/completa-$fecha_semana ]; then
		# Comprimimos la carpeta root para encriptarla y eliminamos la anterior
		cd /root/backups/quijote/completas/completa-$fecha_semana && tar cfz root.tar.gz root && gpg --encrypt --recipient Cabezas root.tar.gz && rm -rf root && rm root.tar.gz
		# Comprimimos la copia de seguridad y eliminamos la carpeta
		cd .. && tar cfz completa-$fecha_semana.tar.gz completa-$fecha_semana && rm -rf completa-$fecha_semana
	fi

	# Hacemos copia completa a quijote y la envíamos a freston
	ssh root@quijote "rsync -tazhe ssh --exclude=$exclude / root@freston:backups/quijote/completas/completa-$fecha_hoy"
	
	# Mandamos correo si la copia ha salido correcta/incorrecta
	fail_com_qui=$?

	if [ $fail_com_qui -eq 0 ]; then
    	mail -s "Copia completa quijote correcta" alejandrocabezab@gmail.com <<< ""
	else
		mail -s "Copia completa quijote incorrecta" alejandrocabezab@gmail.com <<< ""
	fi

	# Guardamos nombre última copia
	echo completa-$fecha_hoy > /root/backups/quijote/completas/latest


	# Ponemos última copia incremental vacía
	echo "" > /root/backups/quijote/incrementales/latest

## FRESTON ##
	## Copia semana anterior ##
	# Si existe dicha copia..

	if [ -d /root/backups/freston/completas/completa-$fecha_semana ]; then
		# Comprimimos la carpeta root para encriptarla y eliminamos la anterior
		cd /root/backups/freston/completas/completa-$fecha_semana && tar cfz root.tar.gz root && gpg --encrypt --recipient Cabezas root.tar.gz && rm -rf root && rm root.tar.gz
		# Comprimimos la copia de seguridad y eliminamos la carpeta
		cd .. && tar cfz completa-$fecha_semana.tar.gz completa-$fecha_semana && rm -rf completa-$fecha_semana
	fi

	# Hacemos copia completa a freston
	rsync -tazh --exclude={"/root/backups","/bin","/usr","/initrd.img.old","/lib64","/media","/proc","/sbin","/tmp","/vmlinuz*","/boot","/lib","/libx32","/mnt","/dev","/initrd.img","/lib32","/lost+found","/opt","/run","/sys"} / /root/backups/freston/completas/completa-$fecha_hoy
	
	# Mandamos correo si la copia ha salido correcta/incorrecta
	fail_com_fre=$?

	if [ $fail_com_fre -eq 0 ]; then
    	mail -s "Copia completa freston correcta" alejandrocabezab@gmail.com <<< ""
	else
		mail -s "Copia completa freston incorrecta" alejandrocabezab@gmail.com <<< ""
	fi

	# Guardamos nombre última copia
	echo completa-$fecha_hoy > /root/backups/freston/completas/latest


	# Ponemos última copia incremental vacía
	echo "" > /root/backups/freston/incrementales/latest

else

	## DULCINEA ##
	# Obtenemos última copia completa e incremental dulcinea
	while IFS= read -r line
	do
	  comp_dulcinea_latest="$line"
	done < /root/backups/dulcinea/completas/latest

	while IFS= read -r line
	do
	  inc_dulcinea_latest="$line"
	done < /root/backups/dulcinea/incrementales/latest

	# Si la última incremental está vacía significa que es Lunes, por ello referenciamos a la completa
	if [[ -z $inc_dulcinea_latest ]]; then
		# Hacemos copia incremental a dulcinea y la envíamos a freston, dicha copia la comparamos con la última completa
		ssh root@dulcinea "rsync -tazhbe ssh --exclude=$exclude --delete --prune-empty-dirs --compare-dest="/root/backups/dulcinea/completas/$comp_dulcinea_latest" / root@freston:backups/dulcinea/incrementales/inc-$fecha_hoy"
		fail_inc_dul=$?
	else
		# Hacemos copia incremental a dulcinea y la envíamos a freston, dicha copia la comparamos con la última incremental y con la última completa
		ssh root@dulcinea "rsync -tazhbe ssh --exclude=$exclude --delete --prune-empty-dirs --compare-dest="/root/backups/dulcinea/incrementales/$inc_dulcinea_latest" --compare-dest="/root/backups/dulcinea/completas/$comp_dulcinea_latest" / root@freston:backups/dulcinea/incrementales/inc-$fecha_hoy"
		fail_inc_dul=$?
	fi

	if [ $fail_inc_dul -eq 0 ]; then
    	mail -s "Copia incremental dulcinea correcta" alejandrocabezab@gmail.com <<< ""
	else
		mail -s "Copia incremental dulcinea incorrecta" alejandrocabezab@gmail.com <<< ""
	fi

	# Guardamos nombre última copia incremental
	echo inc-$fecha_hoy > /root/backups/dulcinea/incrementales/latest
	
	# Última copia incremental de la semana, se comprime si es sábado
	if [[ $(date +%u) -eq 6 ]]; then
		cd /root/backups/dulcinea/incrementales/inc-$fecha_hoy
		if [ -d root ]; then
			# Comprimimos la carpeta root para encriptarla y eliminamos la anterior
			tar cfz root.tar.gz root && gpg --encrypt --recipient Cabezas root.tar.gz && rm -rf root && rm root.tar.gz
		fi
		# Comprimimos la copia de seguridad y eliminamos la carpeta
		cd .. && tar cfz inc-$fecha_hoy.tar.gz inc-$fecha_hoy
		rm -rf inc-$fecha_hoy
	fi

	## Copia día anterior ##
	# Si existe dicha copia..
	if [ -d /root/backups/dulcinea/incrementales/inc-$fecha_dia_menos ]; then
		if [[ $(ls /root/backups/dulcinea/incrementales/inc-$fecha_dia_menos) ]]; then
			cd /root/backups/dulcinea/incrementales/inc-$fecha_dia_menos 
			if [ -d root ]; then
				# Comprimimos la carpeta root para encriptarla y eliminamos la anterior
				tar cfz root.tar.gz root && gpg --encrypt --recipient Cabezas root.tar.gz && rm -rf root && rm root.tar.gz
			fi
			# Comprimimos la copia de seguridad y eliminamos la carpeta
			cd .. && tar cfz inc-$fecha_dia_menos.tar.gz inc-$fecha_dia_menos && rm -rf inc-$fecha_dia_menos
		fi
	fi

	## SANCHO ##
	# Obtenemos última copia completa e incremental sancho
	while IFS= read -r line
	do
	  comp_sancho_latest="$line"
	done < /root/backups/sancho/completas/latest

	while IFS= read -r line
	do
	  inc_sancho_latest="$line"
	done < /root/backups/sancho/incrementales/latest

	# Si la última incremental está vacía significa que es Lunes, por ello referenciamos a la completa
	if [[ -z $inc_sancho_latest ]]; then
		# Hacemos copia incremental a sancho y la envíamos a freston, dicha copia la comparamos con la última completa
		ssh root@sancho "rsync -tazhbe ssh --exclude=$exclude --delete --prune-empty-dirs --compare-dest="/root/backups/sancho/completas/$comp_sancho_latest" / root@freston:backups/sancho/incrementales/inc-$fecha_hoy"
		fail_inc_san=$?
	else
		# Hacemos copia incremental a sancho y la envíamos a freston, dicha copia la comparamos con la última incremental y con la última completa
		ssh root@sancho "rsync -tazhbe ssh --exclude=$exclude --delete --prune-empty-dirs --compare-dest="/root/backups/sancho/incrementales/$inc_sancho_latest" --compare-dest="/root/backups/sancho/completas/$comp_sancho_latest" / root@freston:backups/sancho/incrementales/inc-$fecha_hoy"
		fail_inc_san=$?
	fi

	if [ $fail_inc_san -eq 0 ]; then
    	mail -s "Copia incremental sancho correcta" alejandrocabezab@gmail.com <<< ""
	else
		mail -s "Copia incremental sancho incorrecta" alejandrocabezab@gmail.com <<< ""
	fi

	# Guardamos nombre última copia incremental
	echo inc-$fecha_hoy > /root/backups/sancho/incrementales/latest
	
	# Última copia incremental de la semana, se comprime si es sábado
	if [[ $(date +%u) -eq 6 ]]; then
		cd /root/backups/sancho/incrementales/inc-$fecha_hoy
		if [ -d root ]; then
			# Comprimimos la carpeta root para encriptarla y eliminamos la anterior
			tar cfz root.tar.gz root && gpg --encrypt --recipient Cabezas root.tar.gz && rm -rf root && rm root.tar.gz
		fi
		# Comprimimos la copia de seguridad y eliminamos la carpeta
		cd .. && tar cfz inc-$fecha_hoy.tar.gz inc-$fecha_hoy && rm -rf inc-$fecha_hoy
	fi

	## Copia día anterior ##
	# Si existe dicha copia..
	if [ -d /root/backups/sancho/incrementales/inc-$fecha_dia_menos ]; then
		if [[ $(ls /root/backups/sancho/incrementales/inc-$fecha_dia_menos) ]]; then
			cd /root/backups/sancho/incrementales/inc-$fecha_dia_menos 
			if [ -d root ]; then
				# Comprimimos la carpeta root para encriptarla y eliminamos la anterior
				tar cfz root.tar.gz root && gpg --encrypt --recipient Cabezas root.tar.gz && rm -rf root && rm root.tar.gz
			fi
			# Comprimimos la copia de seguridad y eliminamos la carpeta
			cd .. && tar cfz inc-$fecha_dia_menos.tar.gz inc-$fecha_dia_menos && rm -rf inc-$fecha_dia_menos
		fi
	fi

	## QUIJOTE ##
	# Obtenemos última copia completa e incremental quijote
	while IFS= read -r line
	do
	  comp_quijote_latest="$line"
	done < /root/backups/quijote/completas/latest

	while IFS= read -r line
	do
	  inc_quijote_latest="$line"
	done < /root/backups/quijote/incrementales/latest

	# Si la última incremental está vacía significa que es Lunes, por ello referenciamos a la completa
	if [[ -z $inc_quijote_latest ]]; then
		# Hacemos copia incremental a quijote y la envíamos a freston, dicha copia la comparamos con la última completa
		ssh root@quijote "rsync -tazhbe ssh --exclude=$exclude --delete --prune-empty-dirs --compare-dest="/root/backups/quijote/completas/$comp_quijote_latest" / root@freston:backups/quijote/incrementales/inc-$fecha_hoy"
		fail_inc_qui=$?
	else
		# Hacemos copia incremental a quijote y la envíamos a freston, dicha copia la comparamos con la última incremental y con la última completa
		ssh root@quijote "rsync -tazhbe ssh --exclude=$exclude --delete --prune-empty-dirs --compare-dest="/root/backups/quijote/incrementales/$inc_quijote_latest" --compare-dest="/root/backups/quijote/completas/$comp_quijote_latest" / root@freston:backups/quijote/incrementales/inc-$fecha_hoy"
		fail_inc_qui=$?
	fi

	if [ $fail_inc_qui -eq 0 ]; then
    	mail -s "Copia incremental quijote correcta" alejandrocabezab@gmail.com <<< ""
	else
		mail -s "Copia incremental quijote incorrecta" alejandrocabezab@gmail.com <<< ""
	fi

	# Guardamos nombre última copia incremental
	echo inc-$fecha_hoy > /root/backups/quijote/incrementales/latest
	
	# Última copia incremental de la semana, se comprime si es sábado
	if [[ $(date +%u) -eq 6 ]]; then
		cd /root/backups/quijote/incrementales/inc-$fecha_hoy
		if [ -d root ]; then
			# Comprimimos la carpeta root para encriptarla y eliminamos la anterior
			tar cfz root.tar.gz root && gpg --encrypt --recipient Cabezas root.tar.gz && rm -rf root && rm root.tar.gz
		fi
		# Comprimimos la copia de seguridad y eliminamos la carpeta
		cd .. && tar cfz inc-$fecha_hoy.tar.gz inc-$fecha_hoy && rm -rf inc-$fecha_hoy
	fi

	## Copia día anterior ##
	# Si existe dicha copia..
	if [ -d /root/backups/quijote/incrementales/inc-$fecha_dia_menos ]; then
		if [[ $(ls /root/backups/quijote/incrementales/inc-$fecha_dia_menos) ]]; then
			cd /root/backups/quijote/incrementales/inc-$fecha_dia_menos 
			if [ -d root ]; then
				# Comprimimos la carpeta root para encriptarla y eliminamos la anterior
				tar cfz root.tar.gz root && gpg --encrypt --recipient Cabezas root.tar.gz && rm -rf root && rm root.tar.gz
			fi
			# Comprimimos la copia de seguridad y eliminamos la carpeta
			cd .. && tar cfz inc-$fecha_dia_menos.tar.gz inc-$fecha_dia_menos && rm -rf inc-$fecha_dia_menos
		fi
	fi

	## FRESTON ##
	# Obtenemos última copia completa e incremental freston
	while IFS= read -r line
	do
	  comp_freston_latest="$line"
	done < /root/backups/freston/completas/latest

	while IFS= read -r line
	do
	  inc_freston_latest="$line"
	done < /root/backups/freston/incrementales/latest

	# Si la última incremental está vacía significa que es Lunes, por ello referenciamos a la completa
	if [[ -z $inc_freston_latest ]]; then
		# Hacemos copia incremental a freston y la envíamos a freston, dicha copia la comparamos con la última completa
		rsync -tazhb --exclude={"/var/spool/postfix/active","/root/backups","/bin","/usr","/initrd.img.old","/lib64","/media","/proc","/sbin","/tmp","/vmlinuz*","/boot","/lib","/libx32","/mnt","/dev","/initrd.img","/lib32","/lost+found","/opt","/run","/sys"} --delete --prune-empty-dirs --compare-dest="/root/backups/freston/completas/$comp_freston_latest" / /root/backups/freston/incrementales/inc-$fecha_hoy
		fail_inc_fre=$?
	else
		# Hacemos copia incremental a freston y la envíamos a freston, dicha copia la comparamos con la última incremental y con la última completa
		rsync -tazhb --exclude={"/var/spool/postfix/active","/root/backups","/bin","/usr","/initrd.img.old","/lib64","/media","/proc","/sbin","/tmp","/vmlinuz*","/boot","/lib","/libx32","/mnt","/dev","/initrd.img","/lib32","/lost+found","/opt","/run","/sys"} --delete --prune-empty-dirs --compare-dest="/root/backups/freston/incrementales/$inc_freston_latest" --compare-dest="/root/backups/freston/completas/$comp_freston_latest" / /root/backups/freston/incrementales/inc-$fecha_hoy
		fail_inc_fre=$?
	fi

	if [ $fail_inc_fre -eq 0 ]; then
    	mail -s "Copia incremental freston correcta" alejandrocabezab@gmail.com <<< ""
	else
		mail -s "Copia incremental freston incorrecta" alejandrocabezab@gmail.com <<< ""
	fi

	# Guardamos nombre última copia incremental
	echo inc-$fecha_hoy > /root/backups/freston/incrementales/latest
	
	# Última copia incremental de la semana, se comprime si es sábado
	if [[ $(date +%u) -eq 6 ]]; then
		cd /root/backups/freston/incrementales/inc-$fecha_hoy
		if [ -d root ]; then
			# Comprimimos la carpeta root para encriptarla y eliminamos la anterior
			tar cfz root.tar.gz root && gpg --encrypt --recipient Cabezas root.tar.gz && rm -rf root && rm root.tar.gz
		fi
		# Comprimimos la copia de seguridad y eliminamos la carpeta
		cd .. && tar cfz inc-$fecha_hoy.tar.gz inc-$fecha_hoy && rm -rf inc-$fecha_hoy
	fi

	## Copia día anterior ##
	# Si existe dicha copia..
	if [ -d /root/backups/freston/incrementales/inc-$fecha_dia_menos ]; then
		if [[ $(ls /root/backups/freston/incrementales/inc-$fecha_dia_menos) ]]; then
			cd /root/backups/freston/incrementales/inc-$fecha_dia_menos 
			if [ -d root ]; then
				# Comprimimos la carpeta root para encriptarla y eliminamos la anterior
				tar cfz root.tar.gz root && gpg --encrypt --recipient Cabezas root.tar.gz && rm -rf root && rm root.tar.gz
			fi
			# Comprimimos la copia de seguridad y eliminamos la carpeta
			cd .. && tar cfz inc-$fecha_dia_menos.tar.gz inc-$fecha_dia_menos && rm -rf inc-$fecha_dia_menos
		fi
	fi
fi
~~~