---
layout: post
title: Despliegue cluster kubernetes con kind
tags: [all, kubernetes, kind, Debian, k8s]
---
# Introducción

Buenas, en este post vamos a hacer un despliegue de un clúster de Kubernetes usando una herramienta bastante sencilla llamada Kind. Usaremos únicamente un nodo que mediante contenedores creará los `controller` y `worker`

Veremos como podemos escalar nuestro escenario muy fácilmente modificando únicamente un fichero `.yaml`.

## Escenario a usar

Vamos a usar un escenario con `vagrant` y `kvm` con un único nodo:

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
sudo apt update && sudo apt install docker.io -y
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

Para hacer diferentes pruebas, hemos instalado la utilidad de [kubectl](https://kubernetes.io/docs/tasks/tools/):

~~~
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
~~~

Para que `kubectl` se sincronice con `kind` fácilmente, crearemos un clúster nuevo una vez instalado `kubectl`.

Ahora ya podremos interactuar con nuestro cluster:

~~~
sudo kind create cluster --config=config.yaml
~~~
~~~
sudo kubectl get nodes
NAME                 STATUS   ROLES                  AGE     VERSION
kind-control-plane   Ready    control-plane,master   5m48s   v1.20.2
kind-worker          Ready    <none>                 5m15s   v1.20.2
kind-worker2         Ready    <none>                 5m14s   v1.20.2
~~~

Vamos a desplegar un `nginx` con el siguiente fichero `deployment.yaml`:

~~~
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: default
  labels:
    app: nginx
spec:
  revisionHistoryLimit: 2
  strategy:
    type: RollingUpdate
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - name: http
          containerPort: 80
~~~

Y su servicio `NodePort` a partir del fichero `service.yaml`:

~~~
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: default
spec:
  type: NodePort
  ports:
  - name: http
    port: 80
    targetPort: http
  selector:
    app: nginx
~~~

Hacemos el despliegue de ambos y comprobamos que se han creado correctamente:

~~~
kubectl apply -f deployment.yaml 
deployment.apps/nginx created

kubectl create -f service.yaml 
service/nginx created

sudo kubectl get all
NAME                        READY   STATUS    RESTARTS   AGE
pod/nginx-bdc5c7d65-46qqz   1/1     Running   0          69s
pod/nginx-bdc5c7d65-qltfb   1/1     Running   0          69s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        13m
service/nginx        NodePort    10.96.91.230   <none>        80:30481/TCP   69s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   2/2     2            2           69s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-bdc5c7d65   2         2         2       69s
~~~

Para poder comprobarlo desde la consola, tendremos que obtener la ip del nodo controlador, para ello:

~~~
sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' kind-control-plane
172.18.0.3
~~~

Y con la ip y el puerto obtenido anteriormente podremos obtener la página generada de `nginx`:

~~~
curl 172.18.0.3:30481
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
~~~