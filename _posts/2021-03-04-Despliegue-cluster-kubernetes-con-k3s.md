---
layout: post
title: Despliegue cluster kubernetes con k3s
tags: [all, kubernetes, k8s, Debian, k3s]
---
# Introducción

Buenas, en este post vamos a desplegar un cluster sencillo de kubernetes con la herramienta `k3s`, tendremos un nodo `controller` y dos `worker` y la gestión del cluster la haremos desde nuestra máquina anfitriona.

## Escenario

El escenario que haremos será sencillo como ya hemos dicho anteriormente, usaremos `vagrant` para hacer las tres máquinas con `libvirt`, el fichero `Vagrantfile` es el siguiente:

~~~
Vagrant.configure("2") do |config|

        config.vm.define :controller do |controller|
          controller.vm.box = "generic/debian10"
          controller.vm.hostname = "controller"
          controller.vm.synced_folder ".", "/vagrant", disabled: true
          controller.vm.provider :libvirt do |libvirt|
            libvirt.uri = 'qemu+unix:///system'
            libvirt.host = "debian"
            libvirt.cpus = 1
            libvirt.memory = 1024
          end
        end

        config.vm.define :worker1 do |worker1|
          worker1.vm.box = "generic/debian10"
          worker1.vm.hostname = "worker1"
          worker1.vm.synced_folder ".", "/vagrant", disabled: true
          worker1.vm.provider :libvirt do |libvirt|
            libvirt.uri = 'qemu+unix:///system'
            libvirt.host = "debian"
            libvirt.cpus = 1
            libvirt.memory = 1024
          end
        end

        config.vm.define :worker2 do |worker2|
          worker2.vm.box = "generic/debian10"
          worker2.vm.hostname = "worker2"
          worker2.vm.synced_folder ".", "/vagrant", disabled: true
          worker2.vm.provider :libvirt do |libvirt|
            libvirt.uri = 'qemu+unix:///system'
            libvirt.host = "debian"
            libvirt.cpus = 1
            libvirt.memory = 1024
          end
        end
end
~~~

Para arrancar el escenario:

~~~
vagrant up
~~~

## Instalación de k3s en controller

Una vez arrancado el escenario, entramos en el nodo `controller`:

~~~
vagrant ssh controller
~~~

Y ejecutaremos el siguiente comando para que se haga la instalación automática de `k3s`

~~~
curl -sfL https://get.k3s.io | sh -
~~~

Aparte de `k3s`, también se han instalado las siguiente utilidades:

* kubectl
* crictl
* k3s-killall.sh
* k3s-uninstall.sh

Una vez instalado, podremos obtener información de los nodos:

~~~
sudo kubectl get nodes
     NAME         STATUS   ROLES                  AGE   VERSION
     controller   Ready    control-plane,master   20m   v1.20.4+k3s1
~~~

### Parámetros necesarios para workers

Para hacer la instalación en los workers, necesitaremos dos parámetros de nuestro `controller`:

1. La dirección ip `INTERNAL-IP`:

	~~~
	sudo kubectl get nodes -o wide
	NAME         STATUS   ROLES                  AGE   VERSION        INTERNAL-IP       EXTERNAL-IP   OS-IMAGE                       KERNEL-VERSION    CONTAINER-RUNTIME
	controller   Ready    control-plane,master   71m   v1.20.4+k3s1   192.168.121.22    <none>        Debian GNU/Linux 10 (buster)   4.19.0-14-amd64   containerd://1.4.3-k3s3
	~~~

2. El token que genera `k3s`, alojado en:

	~~~
	sudo cat /var/lib/rancher/k3s/server/node-token
		K105bb06e0ec50dae9f73cc17c3181f8c003a9cb915fbc40adcd8364e027b61c50b::server:5b5a5f9c0b7cda8d05350fbd52e5b712
	~~~

Una vez obtenidos los dos parámetros, pasaremos a la instalación en los dos `worker`

## Instalación k3s en workers

La instalación será de igual forma para las dos máquinas, primero declaramos las variables obtenidas del `controller`, siendo la primera la dirección url y el puerto usado por defecto por `k3s`:

~~~
k3s_url="https://192.168.121.22:6443"
k3s_token="K105bb06e0ec50dae9f73cc17c3181f8c003a9cb915fbc40adcd8364e027b61c50b::server:5b5a5f9c0b7cda8d05350fbd52e5b712"
~~~

Ahora procedemos a la instalación con el script proporcionado por `k3s`:

~~~
curl -sfL https://get.k3s.io | K3S_URL=${k3s_url} K3S_TOKEN=${k3s_token} sh
...
[INFO]  systemd: Starting k3s-agent
~~~

Una vez realizado en los dos workers, si vamos al nodo controller y obtenemos los nodos:

~~~
sudo kubectl get nodes
NAME         STATUS   ROLES                  AGE   VERSION
controller   Ready    control-plane,master   28m   v1.20.4+k3s1
worker1      Ready    <none>                 22m   v1.20.4+k3s1
worker2      Ready    <none>                 20m   v1.20.4+k3s1
~~~

## Gestión del cluster desde máquina anfitriona

Para poder gestionar el cluster desde nuestra máquina, tendremos que irnos al `controller` y copiar el siguiente fichero:

~~~
sudo cat /etc/rancher/k3s/k3s.yaml

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJkekNDQVIyZ0F3SUJBZ0lCQURBS0JnZ3Foa2pPUFFRREFqQWpNU0V3SHdZRFZRUUREQmhyTTNNdGMyVnkKZG1WeUxXTmhRREUyTVRRNE56a3lNakl3SGhjTk1qRXdNekEwTVRjek16UXlXaGNOTXpFd016QXlNVGN6TXpReQpXakFqTVNFd0h3WURWUVFEREJock0zTXRjMlZ5ZG1WeUxXTmhRREUyTVRRNE56a3lNakl3V1RBVEJnY3Foa2pPClBRSUJCZ2dxaGtqT1BRTUJCd05DQUFUWUNsVE1abFhaSm1PR2U3WlJhd09EbE12NFkzL2RpcXFWazBjNVlDUm4KM2dlNXhRb2xjdXFUMlVXLzJ0SmwraWp3M1k4Q3Q0Z2ptWS92TGczSkNvR1ZvMEl3UURBT0JnTlZIUThCQWY4RQpCQU1DQXFRd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBZEJnTlZIUTRFRmdRVTV0N0ZnOFkrTU9IOW1HZ3ppTS9MCjZDNkhGWEV3Q2dZSUtvWkl6ajBFQXdJRFNBQXdSUUlnVXpNS3EzMHp1STRGbXBaeGg1WjZFS1NiWEpyYmlzUWMKcm04akZQRVo0b0lDSVFDNXFDaTAzbjZuVnMrdndpZWJ2RHhsZmYxV3ZLaFVwRy9yWlB2VlNpMDk3UT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://127.0.0.1:6443
  name: default
contexts:
- context:
    cluster: default
    user: default
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: default
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJrVENDQVRlZ0F3SUJBZ0lJWUpUcTNmTk0raDh3Q2dZSUtvWkl6ajBFQXdJd0l6RWhNQjhHQTFVRUF3d1kKYXpOekxXTnNhV1Z1ZEMxallVQXhOakUwT0RjNU1qSXlNQjRYRFRJeE1ETXdOREUzTXpNME1sb1hEVEl5TURNdwpOREUzTXpNME1sb3dNREVYTUJVR0ExVUVDaE1PYzNsemRHVnRPbTFoYzNSbGNuTXhGVEFUQmdOVkJBTVRESE41CmMzUmxiVHBoWkcxcGJqQlpNQk1HQnlxR1NNNDlBZ0VHQ0NxR1NNNDlBd0VIQTBJQUJEaW9Oa1UzOGt0Mmg0dzQKL3VYRVVkZHMxT0VrZ0ZWeUEvTHpGa0xXc2Z3NGxRK08yUFRsOWJaclBXZFM1bGdyUlg5Wmk2aEtWVjNVZkZ6NQo2c3pqWkFtalNEQkdNQTRHQTFVZER3RUIvd1FFQXdJRm9EQVRCZ05WSFNVRUREQUtCZ2dyQmdFRkJRY0RBakFmCkJnTlZIU01FR0RBV2dCUnczSXhBQjdvVFZIUUpPQUtmNG04RDlTTzI1ekFLQmdncWhrak9QUVFEQWdOSUFEQkYKQWlBVk5HZEpyOFcxank0RVYxVFpncDQzN0s4NDlYMVZPeGNWeDN2TFZ1eThOQUloQU5NeHA3dFVUQzlnV1ZkKwpuTWtrZFFwbm1va0tmdVdJSGdMWTRaRGlUZ1l6Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0KLS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJkakNDQVIyZ0F3SUJBZ0lCQURBS0JnZ3Foa2pPUFFRREFqQWpNU0V3SHdZRFZRUUREQmhyTTNNdFkyeHAKWlc1MExXTmhRREUyTVRRNE56a3lNakl3SGhjTk1qRXdNekEwTVRjek16UXlXaGNOTXpFd016QXlNVGN6TXpReQpXakFqTVNFd0h3WURWUVFEREJock0zTXRZMnhwWlc1MExXTmhRREUyTVRRNE56a3lNakl3V1RBVEJnY3Foa2pPClBRSUJCZ2dxaGtqT1BRTUJCd05DQUFSc0kyV2pMTmliTk1oVENwTE5rZ1JPZVhjZU5iVXNzQXFNNGNHcWtDMm8KaWIzTk5WMlpaRWFFT2JzRlhoa1pKU0tpbEFVZkhNSk1RQUx3OGxxdFdsWm9vMEl3UURBT0JnTlZIUThCQWY4RQpCQU1DQXFRd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBZEJnTlZIUTRFRmdRVWNOeU1RQWU2RTFSMENUZ0NuK0p2CkEvVWp0dWN3Q2dZSUtvWkl6ajBFQXdJRFJ3QXdSQUlnTDBtd0FwT3RzQi82QkVYckphekxlK0g5Y2h2ZXF0SWsKVUF2MlFuSjBEWWNDSUYzUitwaHpQeW9ySVRIREV3OUp5ZnZPK3hucWtWWFpHLzhud3VDTHpYMHcKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1IY0NBUUVFSUcyMmJDeHZlbFZRbW5XeXVMbndhcXlBclhjR0FwWXVhZkc0dUlxWmRBekJvQW9HQ0NxR1NNNDkKQXdFSG9VUURRZ0FFT0tnMlJUZnlTM2FIakRqKzVjUlIxMnpVNFNTQVZYSUQ4dk1XUXRheC9EaVZENDdZOU9YMQp0bXM5WjFMbVdDdEZmMW1McUVwVlhkUjhYUG5xek9Oa0NRPT0KLS0tLS1FTkQgRUMgUFJJVkFURSBLRVktLS0tLQo=
~~~

Ahora en nuestra máquina anfitriona, ya instalado `kubectl` previamente, si no lo tenemos instalado, instalamos:

~~~
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
~~~

Copiaremos el fichero a la siguiente ruta, no sin antes cambiar la ip, poniendo la ip de nuestro `controller`:

~~~
~/.kube/config
~~~

Ahora, cargamos el fichero con las credenciales:

~~~
export KUBECONFIG=~/.kube/config
~~~

Y comprobamos que podemos obtener los nodos:

~~~
kubectl get nodes
     NAME         STATUS   ROLES                  AGE   VERSION
     worker2      Ready    <none>                 29m   v1.20.4+k3s1
     controller   Ready    control-plane,master   37m   v1.20.4+k3s1
     worker1      Ready    <none>                 31m   v1.20.4+k3s1
~~~

## Despliegue Letschat

A continuación haremos el despliegue de la aplicación de Letschat. Para ello, tenemos clonado [este](https://github.com/iesgn/kubernetes-storm) repositorio, el cuál tiene bastantes ejemplos y utilización de `kubectl`

Para el despliegue, nos iremos a la siguiente ruta:

~~~
kubernetes-storm/unidad3/ejemplos-3.2/ejemplo8/
~~~

Y ejecutamos:

~~~
kubectl apply -f .
     Warning: networking.k8s.io/v1beta1 Ingress is deprecated in v1.19+, unavailable in v1.22+; use 
     networking.k8s.io/v1 Ingress
     ingress.networking.k8s.io/ingress-letschat created
     deployment.apps/letschat created
     service/letschat created
     deployment.apps/mongo created
     service/mongo created
~~~

Cada fichero desplegará lo siguiente:

* `letschat-deployment`: desplegará un `pod` con la aplicación `letschat` con una replica.

* `mongo-deployment`: desplegará la base de datos necesaria para poder ejecutar `letschat`.

* `letschat-srv`: desplegará el servicio para que la base de datos pueda conectar con `letschat`.

* `mongo-srv`: desplegará el servicio para que la aplicación `letschat` pueda conectar con la base de datos.

* `ingress`: añadirá el componente `ingress` para que se pueda acceder a la aplicación mediante un nombre.

Empezarán a crearse los desliegues:

~~~
kubectl get all,ingress
NAME                            READY   STATUS              RESTARTS   AGE
pod/letschat-7c66bd64f5-wcbkn   0/1     ContainerCreating   0          11s
pod/mongo-5c694c878b-qjtfj      0/1     ContainerCreating   0          11s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/kubernetes   ClusterIP   10.43.0.1       <none>        443/TCP          43m
service/letschat     NodePort    10.43.43.122    <none>        8080:30437/TCP   11s
service/mongo        ClusterIP   10.43.255.141   <none>        27017/TCP        10s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/letschat   0/1     1            0           11s
deployment.apps/mongo      0/1     1            0           11s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/letschat-7c66bd64f5   1         1         0       11s
replicaset.apps/mongo-5c694c878b      1         1         0       11s

NAME                                         CLASS    HOSTS              ADDRESS           PORTS   AGE
ingress.networking.k8s.io/ingress-letschat   <none>   www.letschat.com   192.168.121.213   80      11s
~~~

Y pasados unos segundos, todo estará listo:

~~~
kubectl get all,ingress
NAME                            READY   STATUS    RESTARTS   AGE
pod/mongo-5c694c878b-qjtfj      1/1     Running   0          30s
pod/letschat-7c66bd64f5-wcbkn   1/1     Running   1          30s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/kubernetes   ClusterIP   10.43.0.1       <none>        443/TCP          43m
service/letschat     NodePort    10.43.43.122    <none>        8080:30437/TCP   30s
service/mongo        ClusterIP   10.43.255.141   <none>        27017/TCP        29s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mongo      1/1     1            1           30s
deployment.apps/letschat   1/1     1            1           30s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/mongo-5c694c878b      1         1         1       30s
replicaset.apps/letschat-7c66bd64f5   1         1         1       30s

NAME                                         CLASS    HOSTS              ADDRESS           PORTS   AGE
ingress.networking.k8s.io/ingress-letschat   <none>   www.letschat.com   192.168.121.213   80      30s
~~~

## Escalado

Para comprobar que funciona el escalado, bastaría con ejecutar lo siguiente:

~~~
kubectl scale deployment letschat --replicas=10
~~~

Y comprobamos que se empezarán a escalar las réplicas deseadas:

~~~
kubectl get all  

     NAME                            READY   STATUS              RESTARTS   AGE
     pod/mongo-5c694c878b-qjtfj      1/1     Running             0          17m
     pod/letschat-7c66bd64f5-wcbkn   1/1     Running             1          17m
     pod/letschat-7c66bd64f5-fqzsv   0/1     ContainerCreating   0          12s
     pod/letschat-7c66bd64f5-9brf8   0/1     ContainerCreating   0          12s
     pod/letschat-7c66bd64f5-4lncv   0/1     ContainerCreating   0          12s
     pod/letschat-7c66bd64f5-wsrbg   0/1     ContainerCreating   0          12s
     pod/letschat-7c66bd64f5-mlkfh   0/1     ContainerCreating   0          12s
     pod/letschat-7c66bd64f5-cpv4w   1/1     Running             0          12s
     pod/letschat-7c66bd64f5-6242w   1/1     Running             0          12s
     pod/letschat-7c66bd64f5-p59vr   1/1     Running             0          12s
     pod/letschat-7c66bd64f5-q2sdf   1/1     Running             0          12s

     NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
     service/kubernetes   ClusterIP   10.43.0.1       <none>        443/TCP          60m
     service/letschat     NodePort    10.43.43.122    <none>        8080:30437/TCP   17m
     service/mongo        ClusterIP   10.43.255.141   <none>        27017/TCP        17m

     NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
     deployment.apps/mongo      1/1     1            1           17m
     deployment.apps/letschat   5/10    10           5           17m

     NAME                                  DESIRED   CURRENT   READY   AGE
     replicaset.apps/mongo-5c694c878b      1         1         1       17m
     replicaset.apps/letschat-7c66bd64f5   10        10        5       17m
~~~

Y si volvemos a comprobar al cabo de unos segundos:

~~~
kubectl get all

     NAME                            READY   STATUS    RESTARTS   AGE
     pod/mongo-5c694c878b-qjtfj      1/1     Running   0          19m
     pod/letschat-7c66bd64f5-wcbkn   1/1     Running   1          19m
     pod/letschat-7c66bd64f5-cpv4w   1/1     Running   0          94s
     pod/letschat-7c66bd64f5-6242w   1/1     Running   0          94s
     pod/letschat-7c66bd64f5-p59vr   1/1     Running   0          94s
     pod/letschat-7c66bd64f5-q2sdf   1/1     Running   0          94s
     pod/letschat-7c66bd64f5-4lncv   1/1     Running   0          94s
     pod/letschat-7c66bd64f5-wsrbg   1/1     Running   0          94s
     pod/letschat-7c66bd64f5-9brf8   1/1     Running   0          94s
     pod/letschat-7c66bd64f5-mlkfh   1/1     Running   0          94s
     pod/letschat-7c66bd64f5-fqzsv   1/1     Running   0          94s

     NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
     service/kubernetes   ClusterIP   10.43.0.1       <none>        443/TCP          62m
     service/letschat     NodePort    10.43.43.122    <none>        8080:30437/TCP   19m
     service/mongo        ClusterIP   10.43.255.141   <none>        27017/TCP        19m

     NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
     deployment.apps/mongo      1/1     1            1           19m
     deployment.apps/letschat   10/10   10           10          19m

     NAME                                  DESIRED   CURRENT   READY   AGE
     replicaset.apps/mongo-5c694c878b      1         1         1       19m
     replicaset.apps/letschat-7c66bd64f5   10        10        10      19m
~~~

## Componente ingress

Para comprobar que está funcionando el componente `ingress`, vamos a acceder a la página a través de nuestro navegador, no sin antes añadir a nuestro fichero:

~~~
/etc/hosts
~~~

La siguiente línea:

~~~
192.168.121.22  www.letschat.com
~~~

Y comprobamos:

![1](/assets/img/posts/k3s/1.png)