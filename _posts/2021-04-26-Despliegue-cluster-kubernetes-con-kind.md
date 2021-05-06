---
layout: post
title: Despliegue cluster kubernetes con k3s
tags: [all, kubernetes, kind, Debian, k8s]
---
# Introducción

Buenas, en este post vamos a hacer un despliegue de un clúster de Kubernetes usando una herramienta bastante sencilla llamada Kind.

Veremos como podemos escalar nuestro escenario muy fácilmente modificando únicamente un fichero `.yaml`.

## Escenario a usar

Vamos a usar un escenario con `vagrant`:

~~~
Vagrant.configure("2") do |config|

        config.vm.define :kind do |kind|
          kind.vm.box = "generic/debian10"
          kind.vm.hostname = "kind"
          kind.vm.synced_folder ".", "/vagrant", disabled: true
          kind.vm.provider :libvirt do |libvirt|
            libvirt.uri = 'qemu+unix:///system'
            libvirt.host = "debian"
            libvirt.cpus = 1
            libvirt.memory = 1024
          end
        end

end
~~~

Para levantar el escenario:

~~~
vagrant up
~~~

## Instalación de kind

Entramos en nuestra máquina que hemos creado:

~~~
vagrant ssh kind
~~~

Una vez dentro, antes de instalar `kind`, debemos de instalar `docker`, ya que cada nodo del cluster se va ejecutar en un contenedor, para ello:

~~~
sudo apt update && sudo apt install docker.io
~~~

Ahora procedemos a la instalación de `kind`:

~~~
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.10.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin
~~~

Y ya podemos ver su versión:

~~~
kind version
kind v0.10.0 go1.15.7 linux/amd64
~~~

## Despliegue cluster básico kubernetes

Para crear un cluster de kubernetes básico con un único contenedor con el rol de `controller` y `worker` ejecutaremos la siguiente instrucción:

~~~
sudo kind create cluster
~~~

Y podemos comprobarlo:

~~~
sudo kind get clusters
kind

sudo kind get nodes
kind-control-plane
~~~

Para borrar el cluster:

~~~
sudo kind delete cluster
~~~

## Despliegue cluster más complejos

Sin embargo, queremos crear cluster más complejos, para ello tenemos que crear un fichero `config.yaml` en el cual especificaremos nuestro cluster, por ejemplo:

~~~
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
~~~

Y para crearlo mediante nuestro fichero:

~~~
sudo kind create cluster --config=config.yaml
~~~

Comprobamos:

~~~
sudo kind get nodes
kind-control-plane
kind-worker
kind-worker2
~~~

Y también podríamos comprobarlo a través de `docker`:

~~~
sudo docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                       NAMES
c2abbb8d7d3d        kindest/node:v1.20.2   "/usr/local/bin/entr…"   2 minutes ago       Up 2 minutes        127.0.0.1:39921->6443/tcp   kind-control-plane
c04733ee609c        kindest/node:v1.20.2   "/usr/local/bin/entr…"   2 minutes ago       Up 2 minutes                                    kind-worker
7f711e59a193        kindest/node:v1.20.2   "/usr/local/bin/entr…"   2 minutes ago       Up 2 minutes                                    kind-worker2
~~~

También podríamos crearlos con alta disponibilidad en los controladores, en el fichero `config.yaml`:

~~~
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: control-plane
- role: control-plane
- role: worker
- role: worker
- role: worker
~~~

Y comprobamos:

~~~
sudo kind get nodes
kind-worker2
kind-worker
kind-control-plane3
kind-control-plane2
kind-control-plane
kind-external-load-balancer
kind-worker3
~~~

## Interactuando con el cluster

Para hacer diferentes pruebas, hemos instalado la utilidad de `kubectl`, con la cual podemos comprobar nuestro cluster:

~~~
sudo kubectl get nodes
NAME                  STATUS   ROLES                  AGE   VERSION
kind-control-plane    Ready    control-plane,master   18m   v1.20.2
kind-control-plane2   Ready    control-plane,master   18m   v1.20.2
kind-control-plane3   Ready    control-plane,master   17m   v1.20.2
kind-worker           Ready    <none>                 15m   v1.20.2
kind-worker2          Ready    <none>                 15m   v1.20.2
kind-worker3          Ready    <none>                 15m   v1.20.2
~~~

