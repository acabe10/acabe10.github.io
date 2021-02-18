---
layout: post
title: Cifrado asimétrico con GPG y openSSL
tags: [GPG, OpenSSL, Cifrado]
---
# Introducción

Buenas, en esta práctica vamos a hacer una serie de ejercicios para cifrar ficheros utilizando cifrado asimétrico utilizando el programa gpg y openssl.

## TAREA 1: Generación de claves

Los algoritmos de cifrado asimétrico utilizan dos claves para el cifrado y descifrado de mensajes. Cada persona involucrada (receptor y emisor) debe disponer, por tanto, de una pareja de claves pública y privada. Para generar nuestra pareja de claves con gpg utilizamos la opción --gen-key:

Para esta práctica no es necesario que indiquemos frase de paso en la generación de las claves (al menos para la clave pública).

### Generar claves pública y privada

Para generar un par de claves pública y privada:

~~~
gpg --gen-key
~~~

Para poder generarlas modificando su fecha de validez:

~~~
gpg --full-generate-key
~~~

### Listamos las claves que tenemos en nuestra máquina

Para poder listar las claves públicas que tenemos en nuestra máquina:

~~~
gpg -k
~~~

### Listamos las claves privadas

Para poder listar las claves privadas que tenemos en nuestra máquina:

~~~
gpg --list-secret-keys
~~~

## TAREA 2: Importar/exportar clave pública

Para enviar archivos cifrados a otras personas, necesitamos disponer de sus claves públicas. De la misma manera, si queremos que cierta persona pueda enviarnos datos cifrados, ésta necesita conocer nuestra clave pública. Para ello, podemos hacérsela llegar por email por ejemplo. Cuando recibamos una clave pública de otra persona, ésta deberemos incluirla en nuestro keyring o anillo de claves, que es el lugar donde se almacenan todas las claves públicas de las que disponemos.

### Exportar clave pública

Exportamos la clave pública en formato ASCII:

~~~
gpg --export -a "Alejandro Cabezas Barea" > AlejandroCB.asc
~~~

### Importamos clave pública

Importamos una clave pública de un compañero:

~~~
gpg --import juanluis_millan.asc
~~~

### Comprobamos que se han incluido correctamente

~~~
gpg -k
      /home/arya/.gnupg/pubring.kbx
      -----------------------------
      pub   rsa3072 2019-10-07 [SC] [expires: 2021-10-06]
            9CB2265F6EE2F2148BA2D4E96536BA4F91A44CBF
      uid           [ unknown] Juan Luis Millan Hidalgo <juanluismillanhidalgo@gmail.com>
      sub   rsa3072 2019-10-07 [E] [expires: 2021-10-06]

      pub   rsa1024 2019-10-14 [SC] [expires: 2021-10-13]
            D58A310B0E6085C2BE92E7E360D33503FECA5387
      uid           [ultimate] Alejandro Cabezas Barea <alejandrocabezab@gmail.com>
      sub   rsa1024 2019-10-14 [E] [expires: 2021-10-13]
~~~

## TAREA 3: Cifrado asimétrico con claves públicas

Tras realizar el ejercicio anterior, podemos enviar ya documentos cifrados utilizando la clave pública de los destinatarios del mensaje.

### Ciframos un fichero

Vamos a cifrar un fichero para que lo pueda descifrar únicamente nuestro compañero:

~~~
gpg -e -u "Alejandro Cabezas Barea" -r "Juan Luis Millan Hidalgo" ficheroAC.txt
~~~

### Descifrando un fichero

Desciframos un fichero el cual nuestro compañero ha cifrado para nosotros:

~~~
gpg -d fichero.txt.gpg 
    
gpg: encrypted with 1024-bit RSA key, ID 9ED9A1A8EA4FB791, created 2019-10-14
          "Alejandro Cabezas Barea <alejandrocabezab@gmail.com>" 
Este es un mensaje de Juan Luis Millan Hidalgo
~~~

### Eliminar claves

Para eliminar una clave pública:

~~~
gpg --delete-key "Nombre de Usuario o identificador de la clave"
~~~

Para eliminar una clave privada:

~~~
gpg --delete-secret-keys "Nombre de Usuario o identificador de la clave"
~~~

## TAREA 4: Exportar clave a un servidor público de claves PGP

Para distribuir las claves públicas es mucho más habitual utilizar un servidor específico para distribuirlas, que permite a los clientes añadir las claves públicas a sus anillos de forma mucho más sencilla.

### Generamos clave de revocación

En el caso que haya algún problema y queramos revocar la clave, haremos una clave de revocación, para ello:

~~~
gpg --gen-revoke "Alejandro Cabezas Barea"
~~~

### Exportar clave pública a servidor

Nosotros la vamos a exportar al servidor de gnupg, para ello:

~~~
gpg --keyserver keys.gnupg.net --send-key D58A310B0E6085C2BE92E7E360D33503FECA5387
~~~

### Añadir clave desde servidor

Para añadir una clave pública desde el servidor haremos lo siguiente:

~~~
gpg --recv-keys --keyserver pgp.rediris.es 9CB2265F6EE2F2148BA2D4E96536BA4F91A44CBF
~~~

## TAREA 5: Cifrado asimétrico con openssl

En esta ocasión vamos a cifrar nuestros ficheros de forma asimétrica utilizando la herramienta openssl.

### Generamos un par de claves

Para generar un par de claves ejecutamos la siguiente instrucción:

~~~
openssl genrsa -aes128 -out key.pem 2048
~~~

### Enviar clave a un compañero

Para poder enviar la clave pública a un compañero tendremos que crearla de la siguiente forma para que cada clave esté en un archivo:

~~~
openssl rsa -in key.pem -out key.pub.pem -outform PEM -pubout
~~~

### Cifrando un fichero

Para cifrar un fichero:

~~~
openssl rsautl -pubin -encrypt -in pruebassl.txt -out pruebassl.enc -inkey juanlu.pub.pem
~~~

### Para descifrar un fichero

Suponiendo que nuestro compañero nos ha enviado un fichero cifrado:

~~~
openssl rsautl -decrypt -inkey ale.pri.pem -in holaopenssl.enc -out holaopenssldes.dec
~~~