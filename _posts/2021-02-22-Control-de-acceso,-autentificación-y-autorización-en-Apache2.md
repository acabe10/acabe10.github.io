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

- El servidor manda una respuesta del tipo <code>401 HTTP/1.1 401 Authorization Required</code> con una cabecera <code>WWW-Authenticate</code> al cliente de la forma:
 
 ~~~
 WWW-Authenticate: Basic realm="Palabra de paso"
 ~~~

- El navegador del cliente muestra una ventana emergente preguntando por el nombre de usuario y contraseña y cuando se rellena se manda una petición con una cabecera <code>Authorization</code>:

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

### Funcionamiento autentificación básica

Cuando desde el cliente intentamos acceder a una URL que esta controlada por el método de autentificación de tipo digest:

