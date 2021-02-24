---
layout: post
title: Configurar envío de correo en Wordpress
tags: [all, CMS, PHP, Wordpress, Correo]
---
# Introducción

Buenas, en esta práctica vamos a instalar un servidor de correos y configurar el envío en el CMS Wordpress con Gmail, para ello seguiremos la [práctica que hicimos anteriormente](https://acabe10.github.io/2021-02-21-Instalaci%C3%B3n-CMS-Wordpress/).

## Instalar servidor de correos en el primer nodo

Para instalar el servidor de correo y varias utilidades:

~~~
apt install postfix mailutils libsasl2-2 libsasl2-modules
~~~

Nos pedirá como queremos que funcione el servicio, seleccionamos lo siguiente:

![29](/assets/img/posts/wordpress-correo/29.png)

También podrán preguntarnos el servidor relay que vamos a usar, ponemos lo siguiente:

~~~
[smtp.gmail.com]:587
~~~

Ahora vamos a configurar el servidor de postfix:

~~~
sudo nano /etc/postfix/main.cf
~~~

Confirmamos que la línea de <code>relayhost</code> tiene el servidor de relay de <code>google</code> y al final del archivo añadimos las siguientes líneas:

~~~
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_sasl_tls_security_options = noanonymous
~~~

Ahora proporcionaremos las credenciales de nuestra cuenta de gmail, para ello:

~~~
sudo nano /etc/postfix/sasl/sasl_passwd
~~~

Y añadimos con el siguiente formato:

~~~
[smtp.gmail.com]:587 user@gmail.com:password
~~~

Ahora, para crear los <code>database files</code> necesarios usamos lo siguiente:

~~~
sudo postmap /etc/postfix/sasl/sasl_passwd
~~~

Verificamos que tenemos dos archivos en <code>/etc/postfix/sasl/sasl_passwd</code> y cambiamos sus permisos para mayor seguridad:

~~~
sudo chmod 400 /etc/postfix/sasl/sasl_passwd*
~~~

Reiniciamos el servidor de postfix:

~~~
sudo systemctl restart postfix
~~~

Tenemos que irnos a nuestra cuenta de google y activar el <code>acceso de aplicaciones poco seguras</code>:

![30](/assets/img/posts/wordpress-correo/30.png)

## Configuración de Wordpress para que utilice el servidor SMTP

Para que Wordpress use el servidor SMTP de google, vamos a tener que hacer varias cosas, la primera será instalar un módulo en Wordpress, para ello:

![32](/assets/img/posts/wordpress-correo/32.png)

Hemos buscado el plugin que necesitamos, pero al pulsar instalar no nos ha dejado, así que nos hemos descargado el plugin <code>WP Mail SMTP</code> manualmente desde la página oficial de Wordpress y hemos pulsado en <code>Subir plugin</code>:

![33](/assets/img/posts/wordpress-correo/33.png)

Nos hemos encontrado con la sorpresa de que nos salía el siguiente error:

![34](/assets/img/posts/wordpress-correo/34.png)

Lo cual nos dice que tenemos una limitación para la subida de ficheros, para poder solucionarlo, editamos el fichero:

~~~
nano /etc/php/7.3/apache2/php.ini
~~~

Cambiamos dos líneas en ese fichero asignandole el valor deseado:

~~~
upload_max_filesize = 20M
post_max_size = 40M
~~~

Reiniciamos el servicio de apache2:

~~~
sudo systemctl restart apache2
~~~

Y ya podremos subir e instalar el plugin, activamos:

![35](/assets/img/posts/wordpress-correo/35.png)

Nos vamos a la configuración del plugin:

![35-1](/assets/img/posts/wordpress-correo/35-1.png)

Y seleccionamos Google como servicio de correo:

![36](/assets/img/posts/wordpress-correo/36.png)

Llegados al paso de indicar un ID cliente y una clave secreta copiaremos el <code>URI de redirección</code> a nuestro portapapeles:

![36-1](/assets/img/posts/wordpress-correo/36-1.png)

Tendremos que irnos a nuestra cuenta de google y crear un nuevo proyecto:

![37](/assets/img/posts/wordpress-correo/37.png)

Y tendremos disponible la API para poder gestionar:

![38](/assets/img/posts/wordpress-correo/38.png)

Ahora nos iremos a nuestro proyecto y tendremos que aceptar la pantalla de consentimiento para poder crear un ID de cliente OAuth, seleccionamos Externos ya que sólo nos dejará esa opción.

![39](/assets/img/posts/wordpress-correo/39.png)

Una vez hecho, nos saldŕa algo como esto:

![40](/assets/img/posts/wordpress-correo/40.png)

Ahora toca crear la credencial OAuth, para ello <code>Crear credenciales > ID de cliente de OAuth</code>:

![41](/assets/img/posts/wordpress-correo/41.png)

Seleccionamos tipo de aplicación web y un nombre. Y en URIs la dirección de nuestra página

![42](/assets/img/posts/wordpress-correo/42.png)

También tendremos que pegar el URI que copiamos antes desde Wordpress en URIs de redirección autorizados:

![43](/assets/img/posts/wordpress-correo/43.png)

Con esto obtenemos las credenciales deseadas:

![44](/assets/img/posts/wordpress-correo/44.png)

Y las pegamos en Wordpress:

![45](/assets/img/posts/wordpress-correo/45.png)

Guardaremos y nos pedirá que demos permiso al plugin desde nuestra cuenta de google:

![46](/assets/img/posts/wordpress-correo/46.png)

Acabará con un mensaje diciendonos que se ha enlazado correctamente a nuestra API de proyecto de Google:

![47](/assets/img/posts/wordpress-correo/47.png)

Ahora nos vamos a la pestaña de <code>Correo de prueba</code> y envíamos uno a nuestro correo:

![48](/assets/img/posts/wordpress-correo/48.png)

Y podremos observar que nos ha llegado el correo correctamente:

![49](/assets/img/posts/wordpress-correo/49.png)

Y con esto tendremos nuestro servidor de correos y Wordpress configurado.