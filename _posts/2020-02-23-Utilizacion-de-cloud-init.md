---
layout: post
title: Utilización de cloud init
tags: [all, OpenStack, Toboso, Cloud-init]
---
# Introducción

Buenas, en este post vamos a volver a crear una instancia en OpenStack de nuestro escenario del Toboso usando el mismo volumen, pero cambiado la configuración de `cloud-init`, de manera que la instancia permaneza bien configurada aunque se reinicie.

## Configuraciones necesarias

Vamos a usar a `Sancho`. Antes que nada tenemos que volver a activar el `dhcp` de nuestra red, ya que al crear una nueva máquina queremos que nos coja una ip.

### Generar contraseña para Sancho

Vamos a generar una contraseña con `mkpasswd`, que viene con el paquete `whois`, la cuál usaremos en nuestro archivo `cloud-config.yaml`:

<pre>
mkpasswd --method=SHA-512 --rounds=4096
Contraseña: 
$6$rounds=4096$VnWm7hvw//YedOc$pngcEHsjE94Gb6nMqbOf7LR.TIU.2FkWPtdn5O3gIl/RJhCMV41FAqfeku1U9B0/viO7NF15GUf7AEiMapeIE/
</pre>

### Obtener ID de volumen

Para poder obtener el ID de los volumenes de nuestro proyecto de Openstack, necesitamos añadir al script de la ejecución de nuestro proyecto la siguiente línea:

<pre>
export OS_VOLUME_API_VERSION=2
</pre>

Así ya podremos ejecutar el siguiente comando para obtener los volúmenes:

<pre>
openstack volume list
+--------------------------------------+----------+-----------+------+-----------------------------------+
| ID                                   | Name     | Status    | Size | Attached to                       |
+--------------------------------------+----------+-----------+------+-----------------------------------+
| 8a21e41a-92f7-432c-aa61-da459ebd693a | freston  | in-use    |   10 | Attached to freston on /dev/vda   |
| 9bc2d3b5-6623-4d99-baaa-7d337b692231 |          | available |    2 |                                   |
| 3d2e9e23-7130-41fa-9086-9268e2733737 | quijote  | in-use    |   10 | Attached to quijote on /dev/vda   |
| 49c2596f-8c38-413c-8b0a-dc470ebb1ade | sancho   | available |   10 |                                   |
| 598f13c1-8a55-47eb-95a9-d29eeded32e9 | dulcinea | in-use    |   10 | Attached to dulcinea on /dev/vda  |
+--------------------------------------+----------+-----------+------+-----------------------------------+
</pre>

### Obtener id red de proyecto

Para obtener la red del proyecto:

<pre>
openstack network list
+--------------------------------------+-----------------------------------+----------------------------------------------------------------------------+
| ID                                   | Name                              | Subnets                                                                    |
+--------------------------------------+-----------------------------------+----------------------------------------------------------------------------+
| 14acf1c8-df75-4b94-b40f-700aec119e5f | DMZ de alejandro.cabeza           | 4bc9f91a-ff87-47e3-a516-c45413b338ac                                       |
| 49812d85-8e7a-4c31-baa2-d427692f6568 | ext-net                           | 158bbe3e-3c98-485e-8042-ba6402111ea6, 6218710b-aa05-46f7-b198-7639efe3da95 |
| 8784de88-3168-4889-8938-afedf821c377 | red de alejandro.cabeza           | 63d2b1bb-4d50-48c9-8882-ea4dd5c4b202                                       |
| c0bfcf94-919a-451b-8ab5-d6c46e5e4663 | red interna de alejandro.cabeza   | 46237ca1-5e8e-45b8-bf8a-ef515c9d17bf                                       |
| de22c64c-0c44-491c-8bc4-cb0d2e33cce9 | red interna de alejandro.cabeza_2 | da95436a-9fef-4892-bddc-3ed1f817be55                                       |
+--------------------------------------+-----------------------------------+----------------------------------------------------------------------------+
</pre>

## Fichero de cloud-init

Hemos creado el siguiente fichero con el cual arrancaremos la máquina:

<pre>
#cloud-config
# Instalamos algunos paquetes:
packages:
  - dnsutils
# Configuramos adecuadamente el hostname
preserve_hostname: false
fqdn: sancho.cabezas.gonzalonazareno.org
hostname: sancho
# Realizamos un update y upgrade
package_update: true
package_upgrade: true
# Definimos la zona horaria
timezone: Europe/Madrid
# Crear usuario profesor y añadir claves profesores
users:
  - name: profesor
    sudo: ALL=(ALL) NOPASSWD:ALL
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC3AUDWjyPANntK+qwHmlJihKQZ1H+AGN02k06dzRHmkvWiNgou/VcCgowhMTGR+0I6nWVwgRSWKJEUEaMu1r9rEeL63GRtUSepCWpClHJG1CuySuJKVGtRdUq+/szDntpJnJW207a78hTeQLjQsyPvbOqkbulQG7xTRCycdT3bH2UO4JI2d+341gkOlxSG/stPQ52Dsbfb274oMRom5r5f2apD3wbfxE9A6qwm4m70G9NYS7T3uKgCiXegO/3GTJD4UbK0ylGUamG5obdS5yD8Ib12vRCCXWav23SAj/4f9MzAnXX8U4ATM/du2FHZBiIzWVH12LYvIEZpUIVYKPSf
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCmjoVIoZCx4QFXvljqozXGqxxlSvO7V2aizqyPgMfGqnyl0J9YXo6zrcWYwyWMnMdRdwYZgHqfiiFCUn2QDm6ZuzC4Lcx0K3ZwO2lgL4XaATykVLneHR1ib6RNroFcClN69cxWsdwQW6dpjpiBDXf8m6/qxVP3EHwUTsP8XaOV7WkcCAqfYAMvpWLISqYme6e+6ZGJUIPkDTxavu5JTagDLwY+py1WB53eoDWsG99gmvyit2O1Eo+jRWN+mgRHIxJTrFtLS6o4iWeshPZ6LvCZ/Pum12Oj4B4bjGSHzrKjHZgTwhVJ/LDq3v71/PP4zaI3gVB9ZalemSxqomgbTlnT
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCfk9mRtOHM3T1KpmGi0KiN2uAM6CDXM3WFcm1wkzKXx7RaLtf9pX+KCuVqHdy/N/9d9wtH7iSmLFX/4gQKQVG00jHiGf3ABufWeIpjmHtT1WaI0+vV47fofEIjDDfSZPlI3p5/c7tefHsIAK6GbQn31yepAcFYy9ZfqAh8H/Y5eLpf3egPZn9Czsvx+lm0I8Q+e/HSayRaiAPUukF57N2nnw7yhPZCHSZJqFbXyK3fVQ/UQVBeNS2ayp0my8X9sIBZnNkcYHFLIWBqJYdnu1ZFhnbu3yy94jmJdmELy3+54hqiwFEfjZAjUYSl8eGPixOfdTgc8ObbHbkHyIrQ91Kz
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCohLqkd0h+z1FV2EJVZg/KHQFafOUvgq6v1wMRaF7qPbckHGi5UDL0/IkimzaCmxvjCQhEPvNa7LfcWZqiAgSFuClBNAL5tHuavgV+QXIug4fHqvTjaCbNSv5/w0ik+66+6hK16KulFIIixooGxUdsUoSYRVIDoon3rUi/Mo9rfarsTdlEdq8ie0m+a3beEd7IA+TFuPqoFRxyYoenTn1vDTa+RtQJrVsqzvWsv/K8obFU4XQQ4l4Sts19VkKFcooBrpvnfIVx4Z3QpmeGXWjmUu6QpNnydoCDQglvmSNtsRhIT2h2jvi/QZJhDyJpILEO5FsfjdZvBDXV5SAo77RF
  - name: ubuntu
    sudo: ALL=(ALL) NOPASSWD:ALL
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCohLqkd0h+z1FV2EJVZg/KHQFafOUvgq6v1wMRaF7qPbckHGi5UDL0/IkimzaCmxvjCQhEPvNa7LfcWZqiAgSFuClBNAL5tHuavgV+QXIug4fHqvTjaCbNSv5/w0ik+66+6hK16KulFIIixooGxUdsUoSYRVIDoon3rUi/Mo9rfarsTdlEdq8ie0m+a3beEd7IA+TFuPqoFRxyYoenTn1vDTa+RtQJrVsqzvWsv/K8obFU4XQQ4l4Sts19VkKFcooBrpvnfIVx4Z3QpmeGXWjmUu6QpNnydoCDQglvmSNtsRhIT2h2jvi/QZJhDyJpILEO5FsfjdZvBDXV5SAo77RF
    passwd: $6$rounds=4096$bLTvzyHQoviujN$tA/wvvSNGqZw/Ctl2s7SdM/orhHu8hk297/9A1cd/TKU8m59DmbMeHnvVQ5XuvV3keVBx/PoOtyTSk0VgMuYT.
    lock_passwd: false
bootcmd:
  - apt-get remove --purge ntp -y
  - echo "NTP=es.pool.ntp.org" >> /etc/systemd/timesyncd.conf
  - systemctl restart systemd-timesyncd.service
  - timedatectl set-ntp true
  - ip r del default
  - ip r add default via 10.0.1.10
network:
  version: 1
  config:
  - type: physical
    name: ens3
    subnets:
      - type: dhcp
</pre>

## Creación de instancia

Una vez eliminada la máquina `sancho`, previamente hemos eliminado también el archivo `/etc/cloud/cloud.cfg.d/99-disable-network-config.cfg` que nos desactivaba el cloud-network. Ejecutamos para crear la instancia:

<pre>
openstack server create --volume 49c2596f-8c38-413c-8b0a-dc470ebb1ade --flavor m1.mini --security-group default --key-name cloud --network c0bfcf94-919a-451b-8ab5-d6c46e5e4663 --user-data cloud-init.yaml sancho
+-----------------------------+------------------------------------------------------------------+
| Field                       | Value                                                            |
+-----------------------------+------------------------------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                                           |
| OS-EXT-AZ:availability_zone |                                                                  |
| OS-EXT-STS:power_state      | NOSTATE                                                          |
| OS-EXT-STS:task_state       | scheduling                                                       |
| OS-EXT-STS:vm_state         | building                                                         |
| OS-SRV-USG:launched_at      | None                                                             |
| OS-SRV-USG:terminated_at    | None                                                             |
| accessIPv4                  |                                                                  |
| accessIPv6                  |                                                                  |
| addresses                   |                                                                  |
| adminPass                   | nNHByyRMrWw3                                                     |
| config_drive                |                                                                  |
| created                     | 2020-12-17T23:42:18Z                                             |
| flavor                      | m1.mini (12)                                                     |
| hostId                      |                                                                  |
| id                          | dd514563-893f-4bf1-9a28-281596c9f099                             |
| image                       | N/A (booted from volume)                                         |
| key_name                    | cloud                                                            |
| name                        | sancho                                                           |
| progress                    | 0                                                                |
| project_id                  | 7968089df76945fe9bdca29d3d159277                                 |
| properties                  |                                                                  |
| security_groups             | name='3b27f993-42d9-4e18-b590-334bbd6287bf'                      |
| status                      | BUILD                                                            |
| updated                     | 2020-12-17T23:42:18Z                                             |
| user_id                     | c179ae8775bb4403bc9564b9787ea4a35a4af3954f1a0d1fc1205481e24dd50a |
| volumes_attached            | id='49c2596f-8c38-413c-8b0a-dc470ebb1ade'                        |
+-----------------------------+------------------------------------------------------------------+
</pre>

Así ya tendríamos la instancia creada y lo único que tendríamos que hacer es ponerle una ip estática igual que la que nos ha asignado el DHCP de la red para poder volver a deshabilitarlo y que todo esté como antes.