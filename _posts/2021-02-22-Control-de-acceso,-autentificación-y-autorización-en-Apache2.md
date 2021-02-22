---
layout: post
title: Control de acceso, autentificación y autorización en Apache2
tags: [Apache2, Apache, Control de acceso, Autentificación, Autorización]
---
# Introducción

Buenas, en este post explicaremos el control de acceso, autentificación, autorización en un servidor <code>apache2</code> y pondremos ejemplos en el mismo.

## Control de acceso

El Control de acceso en un servidor web nos permite determinar desde donde podemos acceder a los recursos del servidor.

En apache2.2 se utilizan las siguientes directivas: <code>order, allow y deny</code>. Un buen manual para que quede más claro lo puedes encontrar en este [enlace](http://systemadmin.es/2011/04/la-directiva-order-de-apache). La directiva <code>satisfy</code> controla como el se debe comportar el servidor cuando tenemos autorizaciones de control de acceso (allow, deny,…) y tenemos autorizaciones de usuarios (require).

En apache2.4 se utilizan las siguientes directivas: <code>Require, RequireAll, RequireAny y RequireNone</code>

## Autentificación básica

El servidor web Apache puede acompañarse de distintos módulos para proporcionar diferentes modelos de autenticación. La primera forma que veremos es la más simple. Usamos para ello el módulo de autenticación básica que viene instalada “de serie” con cualquier Apache: <code>mod_auth_basic</code>. La configuración que tenemos que añadir en el fichero de definición del Virtual Host a proteger podría ser algo así:

~~~
<Directory "/var/www/miweb/privado">
    AuthUserFile "/etc/apache2/claves/passwd.txt"
    AuthName "Palabra de paso"
    AuthType Basic
    Require valid-user
</Directory>
~~~

El método de autentificación básica se indica en la directiva <code>AuthType</code>.

* En <code>Directory</code> escribimos el directorio a proteger, que puede ser el raíz de nuestro Virtual Host o un directorio interior a este.
* En <code>AuthUserFile</code> ponemos el fichero que guardará la información de usuarios y contraseñas que debería de estar, como en este ejemplo, en un directorio que no sea visitable desde nuestro Apache. Ahora comentaremos la forma de generarlo.
* Por último, en AuthName personalizamos el mensaje que aparecerá en la ventana del navegador que nos pedirá la contraseña.
* Para controlar el control de acceso, es decir, que usuarios tienen permiso para obtener el recurso utilizamos las siguientes directivas: <code>AuthGroupFile, Require user, Require group</code>.

El fichero de contraseñas se genera mediante la utilidad <code>htpasswd</code>. Su sintaxis es bien sencilla. Para añadir un nuevo usuario al fichero operamos así:

~~~
htpasswd /etc/apache2/claves/passwd.txt carolina

New password:
Re-type new password:
Adding password for user carolina
~~~

Para crear el fichero de contraseñas con la introducción del primer usuario tenemos que añadir la opción -c (create) al comando anterior. Si por error la seguimos usando al incorporar nuevos usuarios borraremos todos los anteriores, así que cuidado con esto. Las contraseñas, como podemos ver a continuación, no se guardan en claro. Lo que se almacena es el resultado de aplicar una función hash:

~~~
josemaria:rOUetcAKYaliE
carolina:hmO6V4bM8KLdw
alberto:9RjyKKYK.xyhk
~~~

Para denegar el acceso a algún usuario basta con que borremos la línea correspondiente al mismo. No es necesario que le pidamos a Apache que vuelva a leer su configuración cada vez que hagamos algún cambio en este fichero de contraseñas.

La principal ventaja de este método es su sencillez. Sus inconvenientes: lo incómodo de delegar la generación de nuevos usuarios en alguien que no sea un administrador de sistemas o de hacer un front-end para que sea el propio usuario quien cambie su contraseña. Y, por supuesto, que dichas contraseñas viajan en claro a través de la red. Si queremos evitar esto último podemos crear una instancia [Apache con SSL](https://blog.unlugarenelmundo.es/2008/09/23/chuletillas-y-viii-apache-2-con-ssl-en-debian/).

### Funcionamiento autentificación básica

Cuando desde el cliente intentamos acceder a una URL que esta controlada por el método de autentificación básico:

1. El servidor manda una respuesta del tipo <code>401 HTTP/1.1 401 Authorization Required</code> con una cabecera <code>WWW-Authenticate</code> al cliente de la forma:
 
	~~~
	WWW-Authenticate: Basic realm="Palabra de paso"
	~~~

2. El navegador del cliente muestra una ventana emergente preguntando por el nombre de usuario y contraseña y cuando se rellena se manda una petición con una cabecera <code>Authorization</code>:

	~~~
	Authorization: Basic am9zZTpqb3Nl
	~~~

En realidad la información que se manda es el nombre de usuario y la contraseña en base 64, que se puede decodificar fácilmente con cualquier [utilidad](https://www.base64decode.org/).

## Autentificación tipo digest

La autentificación tipo digest soluciona el problema de la transferencia de contraseñas en claro sin necesidad de usar SSL. El procedimiento, como veréis, es muy similar al tipo básico pero cambiando algunas de las directivas y usando la utilidad <code>htdigest</code> en lugar de <code>htpassword</code> para crear el fichero de contraseñas. El módulo de autenticación necesario suele venir con Apache pero no habilitado por defecto. Para activarlo usamos la utilidad <code>a2enmod</code> y, a continuación reiniciamos el servidor Apache:

~~~
sudo a2enmod auth_digest
sudo systemctl restart apache2
~~~

Luego incluimos una sección como esta en el fichero de configuración de nuestro Virtual Host:

~~~
<Directory "/var/www/miweb/privado">
     AuthType Digest
     AuthName "dominio"
     AuthUserFile "/etc/claves/digest.txt"
     Require valid-user
</Directory>
~~~

Como vemos, es muy similar a la configuración necesaria en la autenticación básica. La directiva <code>AuthName</code> que en la autenticación básica se usaba para mostrar un mensaje en la ventana que pide el usuario y contraseña, ahora se usa también para identificar un nombre de dominio (realm) que debe de coincidir con el que aparezca después en el fichero de contraseñas. Dicho esto, vamos a generar dicho fichero con la utilidad <code>htdigest</code>:

~~~
htdigest -c /etc/claves/digest.txt dominio josemaria

Adding password for josemaria in realm dominio.
New password:
Re-type new password:
~~~

Al igual que ocurría con htpassword, la opción <code>-c</code> (create) sólo debemos de usarla al crear el fichero con el primer usuario. Luego añadiremos los restantes usuarios prescindiendo de ella. A continuación vemos el fichero que se genera después de añadir un segundo usuario:

~~~
josemaria:dominio:8d6af4e11e38ee8b51bb775895e11e0f
gemma:dominio:dbd98f4294e2a49f62a486ec070b9b8c
~~~

### Funcionamiento autentificación digest

Cuando desde el cliente intentamos acceder a una URL que esta controlada por el método de autentificación de tipo digest:

1. El servidor manda una respuesta del tipo <code>401 HTTP/1.1 401 Authorization Required</code> con una cabecera <code>WWW-Authenticate</code> al cliente de la forma:

	~~~
	 WWW-Authenticate: Digest realm="dominio", 
                  nonce="cIIDldTpBAA=9b0ce6b8eff03f5ef8b59da45a1ddfca0bc0c485", 
                  algorithm=MD5, 
                  qop="auth"
	~~~

2. El navegador del cliente muestra una ventana emergente preguntando por el nombre de usuario y contraseña y cuando se rellena se manda una petición con una cabecera <code>Authorization</code>:

	~~~
	Authorization	Digest username="jose", 
	                 realm="dominio", 
	                 nonce="cIIDldTpBAA=9b0ce6b8eff03f5ef8b59da45a1ddfca0bc0c485",
	                 uri="/digest/", 
	                 algorithm=MD5, 
	                 response="814bc0d6644fa1202650e2c404460a21", 
	                 qop=auth, 
	                 nc=00000001, 
	                 cnonce="3da69c14300e446b"
	~~~

La información que se manda es responde que en este caso esta cifrada usando md5 y que se calcula de la siguiente manera:

* Se calcula el md5 del nombre de usuario, del dominio (realm) y la contraseña, la llamamos HA1.
* Se calcula el md5 del método de la petición (por ejemplo GET) y de la uri a la que estamos accediendo, la llamamos HA2.
* El reultado que se manda es el md5 de HA1, un número aleatorio (nonce), el contador de peticiones (nc), el qop y el HA2.

Una vez que lo recibe el servidor, puede hacer la misma operación y comprobar si la información que se ha enviado es válida, con lo que se permitiría el acceso.

## Ejercicios

Crea un escenario en Vagrant o reutiliza uno de los que tienes en ejercicios anteriores, que tenga un servidor con una red publica, y una privada y un cliente conectada a la red privada. Crea un host virtual <code>departamentos.iesgn.org</code>.

### Escenario de Vagrant

Hemos definido el siguiente escenario en vagrant:

~~~
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.define :server do |server|
    server.vm.box = "debian/buster64"
    server.vm.hostname = "server"
    server.vm.network "public_network",:bridge=>"wlp5s0"
    server.vm.network "private_network", ip: "192.168.100.1",
      virtualbox__intnet: "intranet"
  end
  
  config.vm.define :client1 do |client1|
    client1.vm.box = "debian/buster64"
    client1.vm.hostname = "client1"
    client1.vm.network "private_network", ip:"192.168.100.2",
      virtualbox__intnet: "intranet"
  end

end
~~~

### Creación de virtualhost

Como ya sabemos de ejercicios anteriores:

~~~
sudo apt install apache2
~~~

Desactivamos el sitio por defecto:

~~~
cd /etc/apache2/sites-available/
sudo a2dissite 000-default.conf
~~~

Creamos la configuración del sitio de la siguiente forma:

~~~
sudo nano departamentos.iesgn.conf

 <VirtualHost *:80>
        ServerName www.departamentos.iesgn.org
        DocumentRoot /var/www/departamentos.iesgn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
 </VirtualHost>
~~~

Creamos página html:

~~~
cd /var/www/
mkdir departamentos.iesgn

sudo cp html/index.html departamentos.iesgn
~~~

Activamos el sitio:

~~~
cd /etc/apache2/sites-available/
sudo a2ensite departamentos.iesgn.conf
~~~

Reiniciamos apache2:

~~~
sudo systemctl restart apache2
~~~

Compruebo que se ha generado correctamente:

![1](/assets/img/posts/control-acceso/1.png)

### Acceso a intranet e internet

A la URL <code>departamentos.iesgn.org/intranet</code> sólo se debe tener acceso desde el cliente de la red local, y no se puede acceder desde la anfitriona por la red pública. A la URL <code>departamentos.iesgn.org/internet</code>, sin embargo, sólo se debe tener acceso desde la anfitriona por la red pública, y no desde la red local.

Creamos los dos directorios necesarios:

~~~
sudo mkdir /var/www/departamentos.iesgn/intranet
sudo mkdir /var/www/departamentos.iesgn/internet
~~~

Y añadimos algo para identificarlos:

~~~
# echo "<p>INTRANET</p>" > /var/www/departamentos.iesgn/intranet/index.html
# echo "<p>INTERNET</p>" > /var/www/departamentos.iesgn/internet/index.html
~~~

Modificamos el fichero de configuración de nuestro sitio:

~~~
# nano /etc/apache2/sites-available/departamentos.iesgn.conf
~~~

Y le añadimos dos directory para permitir el acceso sólo desde las IP requeridas:

~~~
<Directory /var/www/intranet/>
                Require ip 192.168.100.2
</Directory>

<Directory /var/www/internet/>
                Require ip 192.168.1.146
</Directory>
~~~

Reiniciamos el servidor:

~~~
# systemctl restart apache2
~~~

### Comprobaciones

Compruebo desde el cliente el acceso a la intranet no sin antes añadir la siguiente línea al fichero <code>/etc/hosts</code>:

~~~
172.22.100.1 departamentos.iesgn.org
~~~

Hemos instalado <code>lynx</code> para poder comprobar:

~~~
# apt install lynx
~~~

Y compruebo:

~~~
lynx departamentos.iesgn.org/intranet
~~~

![2](/assets/img/posts/control-acceso/2.png)

Comprobamos a internet:

~~~
lynx departamentos.iesgn.org/internet
~~~

![3](/assets/img/posts/control-acceso/3.png)

Y ahora comprobamos desde la anfitriona no sin antes añadirle la línea a <code>/etc/hosts</code>:

![4](/assets/img/posts/control-acceso/4.png)

![5](/assets/img/posts/control-acceso/5.png)

### Autentificación básica

Limita el acceso a la URL <code>departamentos.iesgn.org/secreto</code>. Comprueba las cabeceras de los mensajes HTTP que se intercambian entre el servidor y el cliente. ¿Cómo se manda la contraseña entre el cliente y el servidor?

Creamos el directorio <code>secreto</code>:

~~~
sudo mkdir /var/www/departamentos.iesgn/secreto
~~~

Le añadimos algo de contenido:

~~~
echo "<p>SECRETO</p>" > /var/www/departamentos.iesgn/secreto/index.html
~~~

Editamos el fichero de configuración de nuestro sitio:

~~~
nano /etc/apache2/sites-available/departamentos.iesgn.conf
~~~

Y le añadimos lo siguiente:

~~~
<Directory /var/www/departamentos.iesgn/secreto/>
         AuthUserFile "/etc/apache2/claves/passwd.txt"
         AuthName "Password"
         AuthType Basic
         Require valid-user
</Directory>
~~~

Creamos el directorio donde guardaremos las claves:

~~~
mkdir /etc/apache2/claves
~~~

Generamos el nuevo fichero de claves (opción -c para la primera vez):

~~~
htpasswd -c /etc/apache2/claves/passwd.txt ale
New password: 
Re-type new password: 
Updating password for user ale
~~~

Reiniciamos el servicio:

~~~
# systemctl restart apache2
~~~

Y comprobamos que nos pide la contraseña:

![6](/assets/img/posts/control-acceso/6.png)

Y si introducimos el usuario que hemos creado antes y su contraseña:

![7](/assets/img/posts/control-acceso/7.png)

Mientras hemos accedido, hemos dejado a nuestro servidor escuchando con <code>tcpdump</code>:

~~~
tcpdump -i eth1
~~~

Y nos ha capturado la siguiente petición de la página <code>secreto</code>:

~~~
GET /secreto HTTP/1.1
	Host: departamentos.iesgn.org
	Connection: keep-alive
	Upgrade-Insecure-Requests: 1
	User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.116 Safari/537.36
	Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
	Accept-Encoding: gzip, deflate
	Accept-Language: es,es-ES;q=0.9,en;q=0.8
~~~

Y seguidamente al ingresar el usuario y la contraseña, nos ha capturado lo siguiente:

~~~
GET /secreto HTTP/1.1
	Host: departamentos.iesgn.org
	Connection: keep-alive
	Cache-Control: max-age=0
	Authorization: Basic YWxlOmFsZQ==
	Upgrade-Insecure-Requests: 1
	User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.116 Safari/537.36
	Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
	Accept-Encoding: gzip, deflate
	Accept-Language: es,es-ES;q=0.9,en;q=0.8
~~~

Que como podemos observar, en la línea de <code>Authorization</code> tenemos la contraseña mandada en <code>Base64</code>, la cuál podemos desencriptar con cualquier programa que traduzca <code>Base64</code>.

### Autentificación digest

Cómo hemos visto la autentificación básica no es segura, modifica la autentificación para que sea del tipo digest, y sólo sea accesible a los usuarios pertenecientes al grupo directivos.

Procedemos a hacerlo con <code>digest</code>, lo primero será activar su módulo:

~~~
sudo a2enmod auth_digest
~~~

Modificamos el directory de nuestro fichero de configuración de esta manera:

~~~
<Directory /var/www/departamentos.iesgn/secreto/>
     AuthType Digest
     AuthName "directivos"
     AuthUserFile "/etc/apache2/claves/digest.txt"
     Require valid-user
</Directory>
~~~

Creamos el archivo de contraseñas, directivos es el <code>AuthName</code> del anterior fichero, el cual sólo podrán acceder los usuarios que pertenezcan a ese dominio:

~~~
htdigest -c /etc/apache2/claves/digest.txt directivos aledigest
~~~

Reiniciamos el servicio:

~~~
# systemctl restart apache2
~~~

Probamos a entrar:

![8](/assets/img/posts/control-acceso/8.png)

Y si introducimos correctamente la contraseña:

![9](/assets/img/posts/control-acceso/9.png)

### Combinación control de acceso y autentificación

Vamos a configurar el virtual host para que se comporte de la siguiente manera: el acceso a la URL <code>departamentos.iesgn.org/secreto</code> se hace forma directa desde la <code>intranet</code>, desde la red pública te pide la autentificación.

Para combinar dichas propuestas anteriores, tendremos que cambiar el fichero de configuración de nuestro sitio:

~~~
sudo nano /etc/apache2/sites-available/departamentos.iesgn.conf
~~~

Y le vamos a añadir lo siguiente:

~~~
<Directory /var/www/departamentos.iesgn/secreto/>
                <RequireAny>
                        <RequireAll>
                                Require ip 192.168.100.2
                        </RequireAll>
                        <RequireAll>
                                Require ip 192.168.1.146
                                AuthUserFile "/etc/apache2/claves/digest.txt"
                                AuthName "directivos"
                                AuthType Digest
                                Require valid-user
                        </RequireAll>
                </RequireAny>
</Directory>
~~~

Lo que queremos decir en el fichero anterior es que permitirá entrar si se cumplen alguna de las dos cosas:

1. Si entran a través de la IP 192.168.100.2, que es la de nuestra red local, permitirá el acceso sin autenticación.
2. Si entran a través de la IP 192.168.1.146, que es la de nuestra máquina anfitriona, pedirá autenticación.

### Comprobaciones

Desde la red local:

~~~
lynx departamentos.iesgn.org/secreto
~~~

![10](/assets/img/posts/control-acceso/10.png)

Desde la máquina anfitriona nos pedirá autenticación:

![11](/assets/img/posts/control-acceso/11.png)

Y si introducimos el usuario del grupo directivos:

![12](/assets/img/posts/control-acceso/12.png)