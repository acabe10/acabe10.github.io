---
layout: post
title: Compilación kérnel linux a medida
tags: [Kérnel, Linux, Compilación, Compilar]
---
# Introducción

Buenas, en este post vamos a compilar un kérnel linux a medida de nuestro ordenador. Al ser linux un kérnel libre, es posible descargar el código fuente, configurarlo y comprimirlo. Además, esta tarea a priori compleja, es más sencilla de lo que parece gracias a las herramientas disponibles.

En esta tarea vamos a tratar de compilar un kérnel completamente funcional que reconozca todo el hardware básico de nuestro equipo y que sea a la vez lo más pequeño posible, es decir que incluya un <code>vmlinuz</code> lo más pequeño posible y que incorpore sólo los módulos imprescindibles.

## Requisitos necesarios

Necesitaremos saber que versión de kérnel tenemos, para obtener la versión de kérnel de nuestra máquina:

~~~
uname -r
~~~

Actualizamos el equipo porque vamos a descargar el código fuente de nuestro kérnel de los repositorios y buscamos el paquete:

~~~
# apt update && apt upgrade
~~~

Y buscamos el paquete correspondiente:

~~~
# apt policy linux-source

linux-source:
  Instalados: (ninguno)
  Candidato:  4.19+105+deb10u7
  Tabla de versión:
     5.8.10-1~bpo10+1 100
        100 http://deb.debian.org/debian buster-backports/main amd64 Packages
     4.19+105+deb10u7 500
        500 http://security.debian.org/debian-security buster/updates/main amd64 Packages
        100 /var/lib/dpkg/status
     4.19+105+deb10u6 500
        500 http://deb.debian.org/debian buster/main amd64 Packages
~~~

Como podemos comprobar no está instalado ninguno, así que como ya hemos visto cuál es la versión de nuestro kérnel, instalaremos el correspondiente. También instalaremos el paquete sugerido de <code>qtbase5-dev</code>, que nos servirá para poder hacer esa configuración en el ḱérnel gráficamente.

~~~
sudo apt install linux-source=4.19+105+deb10u7 qtbase5-dev
~~~

Comprobamos que se ha instalado:

~~~
dpkg -l | grep linux-source

ii  linux-source                          4.19+105+deb10u7                             all          Linux kernel source (meta-package)
ii  linux-source-4.19                     4.19.152-1                                   all          Linux kernel source for version 4.19 with Debian patches
~~~

Y mostramos su contenido para ver dónde se nos ha descargado:

~~~
dpkg -L linux-source-4.19
/.
/usr
/usr/share
/usr/share/doc
/usr/share/doc/linux-source-4.19
/usr/share/doc/linux-source-4.19/changelog.Debian.gz
/usr/share/doc/linux-source-4.19/copyright
/usr/src
/usr/src/linux-patch-4.19-rt.patch.xz
/usr/src/linux-source-4.19.tar.xz
~~~

Como vemos se encuentra en <code>/usr/src/</code>. No es recomendable hacer la compilación del kérnel con el usuario root, así que vamos a crear una carpeta en nuestro directorio de usuario:

~~~
mkdir /home/ale/kernel
~~~

Y descomprimimos el fichero del kérnel:

~~~
tar xf /usr/src/linux-source-4.19.tar.xz /home/ale/kernel
~~~

Y ya tendremos el código fuente del kérnel. Podremos obtener las opciones que podemos hacer con <code>make</code> haciendo lo siguiente:

~~~
$ make help
~~~

Las más usadas son las siguientes:

~~~
make clean // Limpiará los archivos que se han obtenido al compilar excepto el archivo de configuración

make mrproper // Elimina también los archivos de configuración.

make config // Decidiremos línea a línea los parámetros que queremos que se incluyan en el kérnel.

make oldconfig // Usa el fichero de configuración de nuestro kérnel actual y nos preguntará si no está seguro, los cambios en la configuración instalada.

make localmodconfig // Comprueba de la lista completa de módulos que tenemos en el kérnel y suprime los que no están en uso.

make xconfig // Asistente gráfico que hemos instalado anteriormente.
~~~

Podemos partir del archivo de configuración de nuestra máquina:

~~~
# cp /boot/config-4.19.0-12-amd64 .config
~~~

Y si hacemos un <code>make oldconfig</code> nos sobreescribirá el fichero <code>.config</code> que acabamos de copiar.

Aunque no es nuestro objetivo, ya podríamos compilar dicho kérnel con el siguiente comando en un paquete <code>.deb</code> para que fácilmente pudiéramos instalarlo en nuestra máquina, vamos a probar:

~~~
$ make bindeb-pkg
~~~

Nos sale el siguiente error, el cual nos dice que nos faltan dependencias que debemos de instalar

~~~
dpkg-checkbuilddeps: fallo: Unmet build dependencies: libelf-dev:native libssl-dev:native
~~~

Para solucionar el problema, tendremos que instalar dichas dependencias:

~~~
# apt install libelf-dev libssl-dev
~~~

Y volvemos a ejecutar:

~~~
$ make bindeb-pkg
~~~

Si miramos el tamaño del paquete <code>.deb</code> que contiene el kérnel:

~~~
~/kernel$ du -hs linux-image-4.19.152_4.19.152-1_amd64.deb 
13M	linux-image-4.19.152_4.19.152-1_amd64.deb
~~~

Así ya tendríamos nuestro paquete <code>.deb</code>, pero lo que queremos es hacerlo mucho menos pesado.

## Compilación a medida

Para hacerlo menos pesado, usaremos la opción <code>localmodconfig</code>, la cuál nos hace una primera reducción de módulos con los que está usando nuestra máquina en este momento. Primero borramos con <code>clean</code> el rastro antiguo:

~~~
$ make clean
~~~

Y generamos el <code>.config</code>:

~~~
$ make localmodconfig
~~~

Mostramos los módulos estáticos y dinámicos que tenemos al comienzo:

~~~
grep "=y" .config|wc -l
1438

grep "=m" .config|wc -l
214
~~~

Nuestro objetivo será reducirlo lo máximo posible quitándole funcionalidades. Para empezar, sobre el archivo de configuración haremos:

~~~
$ make xconfig
~~~

Y después de quitar todos los siguientes módulos:

~~~
CONFIG_DEBUG_INFO
CONFIG_ENABLE_MUST_CHECK
CONFIG_STRIP_ASM_SYMS
CONFIG_PAGE_POISONING
CONFIG_DEBUG_MEMORY_INIT

CONFIG_HARDLOCKUP_DETECTOR
CONFIG_SOFTLOCKUP_DETECTOR
CONFIG_DETECT_HUNG_TASK

CONFIG_FTRACE

CONFIG_FUNCTION_GRAPH_TRACER
CONFIG_FTRACE_SYSCALLS
CONFIG_TRACER_SNAPSHOT
CONFIG_STACK_TRACER
CONFIG_BLK_DEV_IO_TRACE
CONFIG_KPROBE_EVENTS

RUNTIME_TESTING_MENU

CONFIG_UPROBE_EVENTS
CONFIG_TRACING_EVENTS_GPIO

CONFIG_CRYPTO_FIPS
CONFIG_CRYPTO_CRC32C_INTEL
CONFIG_CRYPTO_CRC32_PCLMUL
CONFIG_CRYPTO_CRCT10DIF_PCLMUL
CONFIG_CRYPTO_GHASH_CLMUL_NI_INTEL
CONFIG_CRYPTO_AES_NI_INTEL
CONFIG_CRYPTO_ANSI_CPRNG

CONFIG_CRYPTO_HW
CONFIG_SIGNED_PE_FILE_VERIFICATION
CONFIG_SECONDARY_TRUSTED_KEYRING
CONFIG_SYSTEM_BLACKLIST_KEYRING
CONFIG_EFI_SIGNATURE_LIST_PARSER

XZ_DEC_X86
CONFIG_IRQ_POLL

CONFIG_FIRMWARE_MEMMAP
CONFIG_DMIID
CONFIG_DMI_SYSFS

CONFIG_EFI_VARS
CONFIG_EFI_RUNTIME_MAP
CONFIG_APPLE_PROPERTIES

CONFIG_VIRTUALIZATION

CONFIG_KPROBES
CONFIG_JUMP_LABEL
CONFIG_STACKPROTECTOR--------
	CONFIG_STACKPROTECTOR_STRONG
CONFIG_VMAP_STACK
CONFIG_REFCOUNT_FULL

CONFIG_MODULE_SIG-----
	MODULE_SIG_SHA256

CONFIG_SOUND
CONFIG_USB_SUPPORT

CONFIG_MMC
CONFIG_MEMSTICK
CONFIG_NEW_LEDS
CONFIG_ACCESSIBILITY


CONFIG_VIRT_DRIVERS
VIRTIO_MENU
CONFIG_XEN_BALLOON
CONFIG_XEN_BACKEND
CONFIG_XEN_SYS_HYPERVISOR
CONFIG_XEN_MCE_LOG

CONFIG_STAGING

CONFIG_CHROME_PLATFORMS
CONFIG_PM_DEVFREQ
CONFIG_PWM

CONFIG_POWERCAP

CONFIG_ANDROID

CONFIG_CC_OPTIMIZE_FOR_SIZE
CONFIG_HIGH_RES_TIMERS

CONFIG_NETFILTER
CONFIG_BT

CONFIG_SCHED_MC
CONFIG_HYPERVISOR_GUEST

CONFIG_MACINTOSH_DRIVERS
CONFIG_HWMON

CONFIG_VM_EVENT_COUNTERS

CONFIG_RFKILL
CONFIG_WLAN
CONFIG_WAN

CONFIG_ZONE_DMA
CONFIG_SMP
CONFIG_DMI
CONFIG_CALGARY_IOMMU
CONFIG_X86_16BIT

CONFIG_MEMTEST

MEMORY_HOTPLUG
CONFIG_MEM_SOFT_DIRTY
CONFIG_ARCH_RANDOM
CONFIG_ACPI_AC
CONFIG_ACPI_BATTERY

CONFIG_IPV6
CONFIG_HAMRADIO
WIRELESS

CONFIG_X86_MCE_AMD
CONFIG_X86_SMAP
CONFIG_IP_MROUTE
CONFIG_NETWORK_FILESYSTEMS
CONFIG_SWAP

CONFIG_NAMESPACES
CONFIG_ACPI_EXTLOG
CONFIG_HOTPLUG_PCI_ACPI

CONFIG_FUSION
CONFIG_NET_FC
CONFIG_BSD_PROCESS_ACCT
CONFIG_TASKSTATS
CONFIG_SFI
CONFIG_CPU_FREQ

CONFIG_ISDN
CONFIG_WATCHDOG

CONFIG_AGP

CONFIG_GPIO_SYSFS

CONFIG_NET_SCHED
CONFIG_MSI_WMI
SCSI_LOWLEVEL

CONFIG_INTEL_IDLE
CONFIG_X86_MCE
CONFIG_SYSVIPC
CONFIG_POSIX_MQUEUE
CONFIG_CROSS_MEMORY_ATTACH
CONFIG_PROFILING

CHECKPOINT_RESTORE
CONFIG_FDDI
CONFIG_HIPPI
CONFIG_INPUT_JOYDEV
CONFIG_INPUT_JOYSTICK
CONFIG_INPUT_TABLET
CONFIG_INPUT_TOUCHSCREEN
CONFIG_INPUT_MISC
CONFIG_INPUT_MOUSE
CONFIG_INPUT_MOUSEDEV
DM_UEVENT
CONFIG_ACORN_PARTITION_RISCIX
CONFIG_SECURITY_SELINUX_AVC_STATS
CONFIG_SECURITY_SELINUX_DEVELOP
CONFIG_SERIAL_8250_EXTENDED
CONFIG_EDAC
CONFIG_RTC_CLASS

CONFIG_ACPI_THERMAL
CONFIG_VGA_CONSOLE
CONFIG_CRYPTO_CCM
CONFIG_SECURITY_SELINUX
CONFIG_SECURITY_APPARMOR
CONFIG_SECURITY_TOMOYO
CONFIG_CRYPTO_SHA256 a módulo
CONFIG_CRYPTO_AES_X86_64
CONFIG_CRYPTO_ARC4
CONFIG_PRINTK
CONFIG_PRINT_QUOTA_WARNING
CONFIG_QUOTA
CONFIG_QUOTA_NETLINK_INTERFACE
CONFIG_XFS_QUOTA

CONFIG_HOTPLUG_PCI
CONFIG_IA32_EMULATION
CONFIG_X86_X32

CONFIG_BLK_DEV
CONFIG_SCSI_DH

CONFIG_REGULATOR
CONFIG_DRM_LEGACY
CONFIG_DMADEVICES
CONFIG_IOMMU_SUPPORT

CONFIG_FB_VESA
CONFIG_FB_EFI
CONFIG_FB_MODE_HELPERS

CONFIG_FIRMWARE_EDID

CONFIG_FB_TILEBLITTING

CONFIG_FRAMEBUFFER_CONSOLE
CONFIG_VGA_CONSOLE ACTIVO

CONFIG_FB a modulo

CONFIG_NVMEM
CONFIG_GENERIC_PHY

CONFIG_VGA_SWITCHEROO
CONFIG_VGA_ARB
CONFIG_DRM_DP_CEC

CONFIG_DRM_NOUVEAU
CONFIG_DRM_I915_CAPTURE_ERROR

CONFIG_X86_PLATFORM_DEVICES

CONFIG_BLK_DEV_MD
CONFIG_IPMI_HANDLER
CONFIG_SPI

CONFIG_AUDIT
CONFIG_GART_IOMMU
CONFIG_X86_VSYSCALL_EMULATION

CONFIG_PARTITION_ADVANCED

CONFIG_NET_VENDOR_3COM
CONFIG_NET_VENDOR_ADAPTEC
CONFIG_NET_VENDOR_AGERE
CONFIG_NET_VENDOR_ALACRITECH
CONFIG_NET_VENDOR_ALTEON
CONFIG_NET_VENDOR_AMAZON
CONFIG_NET_VENDOR_AMD
CONFIG_NET_VENDOR_AQUANTIA
CONFIG_NET_VENDOR_BROADCOM
CONFIG_NET_VENDOR_BROCADE
CONFIG_NET_VENDOR_CADENCE
CONFIG_NET_VENDOR_CAVIUM
CONFIG_NET_VENDOR_CHELSIO
CONFIG_NET_VENDOR_CISCO
CONFIG_NET_VENDOR_CORTINA
CONFIG_NET_VENDOR_DEC
CONFIG_NET_VENDOR_DLINK
CONFIG_NET_VENDOR_EMULEX
CONFIG_NET_VENDOR_EZCHIP
CONFIG_NET_VENDOR_HP
CONFIG_NET_VENDOR_HUAWEI
CONFIG_NET_VENDOR_I825XX
CONFIG_NET_VENDOR_INTEL
CONFIG_NET_VENDOR_MARVELL
CONFIG_NET_VENDOR_MELLANOX
CONFIG_NET_VENDOR_MICREL
CONFIG_NET_VENDOR_MICROSEMI
CONFIG_NET_VENDOR_MYRI
CONFIG_NET_VENDOR_NATSEMI
CONFIG_NET_VENDOR_NETERION
CONFIG_NET_VENDOR_NETRONOME
CONFIG_NET_VENDOR_NI
CONFIG_NET_VENDOR_8390
CONFIG_NET_VENDOR_NVIDIA
CONFIG_NET_VENDOR_OKI
CONFIG_NET_VENDOR_PACKET_ENGINES
CONFIG_NET_VENDOR_QLOGIC
CONFIG_NET_VENDOR_RDC
CONFIG_NET_VENDOR_REALTEK
CONFIG_NET_VENDOR_RENESAS
CONFIG_NET_VENDOR_ROCKER
CONFIG_NET_VENDOR_SAMSUNG
CONFIG_NET_VENDOR_SOLARFLARE
CONFIG_NET_VENDOR_SILAN
CONFIG_NET_VENDOR_SIS
CONFIG_NET_VENDOR_SMSC
CONFIG_NET_VENDOR_SOCIONEXT
CONFIG_NET_VENDOR_STMICRO
CONFIG_NET_VENDOR_SUN
CONFIG_NET_VENDOR_SYNOPSYS
CONFIG_NET_VENDOR_TEHUTI
CONFIG_NET_VENDOR_TI
CONFIG_NET_VENDOR_VIA
CONFIG_NET_VENDOR_WIZNET

CONFIG_HFS_FS
CONFIG_HFSPLUS_FS
CONFIG_MINIX_FS
CONFIG_QNX4FS_FS
CONFIG_JFS_FS
CONFIG_XFS_FS
CONFIG_BTRFS_FS

CONFIG_PINCTRL_AMD
CONFIG_PINCTRL_BAYTRAIL
CONFIG_PINCTRL_CHERRYVIEW
CONFIG_PINCTRL_BROXTON
CONFIG_PINCTRL_CANNONLAKE
CONFIG_PINCTRL_CEDARFORK
CONFIG_PINCTRL_DENVERTON
CONFIG_PINCTRL_GEMINILAKE
CONFIG_PINCTRL_ICELAKE
CONFIG_PINCTRL_LEWISBURG
CONFIG_PINCTRL_SUNRISEPOINT
CONFIG_GPIOLIB
~~~

Y al compilar con los procesadores de nuestra máquina y con la opción <code>INSTALL_MOD_STRIP=1</code>, la cuál reducirá el tamaño al depurar los módulos:

~~~
make -j 4 INSTALL_MOD_STRIP=1 bindeb-pkg
~~~

Obtenemos los siguientes:

~~~
m=91
y=772
~~~

Si comparamos los dos <code>vmlinux</code>(antiguo y nuevo):

~~~
5,1M	/boot/vmlinuz-4.19.0-12-amd64
2,6M	/boot/vmlinuz-4.19.152
~~~

Vemos que su tamaño ha disminuido considerablemente.