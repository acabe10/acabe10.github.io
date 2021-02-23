---
layout: post
title: Instalación y configuración servidor correo en OVH
tags: [all, Servidor correo, OVH]
---
# Introducción

Buenas, en este post vamos a instalar y configurar de manera adecuada el servidor de correos en una máquina que tenemos en OVH con el dominio `iesgn02.es`. El nombre del servicio será `mail.iesgn02.es`

## Instalación

Hemos instalado el servicio de `postfix` en nuestro servidor de OVH:

~~~
sudo apt install postfix
~~~

- Servidor tipo `Internet Site`.
- Mailname: `iesgn02.es`.

Hemos configurado un registro MX en nuestro servidor:

~~~
iesgn02.es.
0	MX	10 mail.iesgn02.es.
~~~

Y si hacemos una consulta:

~~~
dig MX iesgn02.es

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> MX iesgn02.es
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 23648
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;iesgn02.es.			IN	MX

;; ANSWER SECTION:
iesgn02.es.		3600	IN	MX	10 mail.iesgn02.es.

;; Query time: 31 msec
;; SERVER: 212.230.135.1#53(212.230.135.1)
;; WHEN: mié feb 10 18:03:53 CET 2021
;; MSG SIZE  rcvd: 60
~~~

## Envío desde local al exterior

Para enviar una prueba de correo:

~~~
mail -s "prueba desde ovh" alejandrocabezab@gmail.com <<< ""
~~~

Mostramos el log de OVH:

~~~
Feb 10 12:29:24 ned postfix/postfix-script[879]: starting the Postfix mail system
Feb 10 12:29:24 ned postfix/master[891]: daemon started -- version 3.4.14, configuration /etc/postfix
Feb 10 18:08:05 ned postfix/pickup[3554]: B08B1A107C: uid=1000 from=<debian>
Feb 10 18:08:05 ned postfix/cleanup[3912]: B08B1A107C: message-id=<20210210170805.B08B1A107C@ned.iesgn02.es>
Feb 10 18:08:05 ned postfix/qmgr[898]: B08B1A107C: from=<debian@iesgn02.es>, size=405, nrcpt=1 (queue active)
Feb 10 18:08:06 ned postfix/smtp[3914]: connect to gmail-smtp-in.l.google.com[2a00:1450:400c:c0a::1a]:25: Network is unreachable
Feb 10 18:08:06 ned postfix/smtp[3914]: B08B1A107C: to=<alejandrocabezab@gmail.com>, relay=gmail-smtp-in.l.google.com[64.233.166.26]:25, delay=1.2, delays=0.02/0.01/0.43/0.77, dsn=2.0.0, status=sent (250 2.0.0 OK  1612976886 196si6190443wma.0 - gsmtp)
Feb 10 18:08:06 ned postfix/qmgr[898]: B08B1A107C: removed
~~~

Captura del correo recibido:

![1](/assets/img/posts/correo/1.png)

Log del correo recibido:

~~~
ARC-Authentication-Results: i=1; mx.google.com;
       spf=pass (google.com: domain of debian@iesgn02.es designates 146.59.196.83 as permitted sender) smtp.mailfrom=debian@iesgn02.es
Return-Path: <debian@iesgn02.es>
Received: from ned.iesgn02.es (vps-8525915c.vps.ovh.net. [146.59.196.83])
        by mx.google.com with ESMTP id x2si4043964wmc.116.2021.02.04.01.17.35
        for <alejandrocabezab@gmail.com>;
        Thu, 04 Feb 2021 01:17:35 -0800 (PST)
Received-SPF: pass (google.com: domain of debian@iesgn02.es designates 146.59.196.83 as permitted sender) client-ip=146.59.196.83;
Authentication-Results: mx.google.com;
       spf=pass (google.com: domain of debian@iesgn02.es designates 146.59.196.83 as permitted sender) smtp.mailfrom=debian@iesgn02.es
Received: by ned.iesgn02.es (Postfix, from userid 1000) id 9877DA1000; Thu,
  4 Feb 2021 10:17:34 +0100 (CET)
To: alejandrocabezab@gmail.com
Subject: prueba desde ovh
~~~

Registro SPF en DNS de OVH:

~~~
iesgn02.es.
0	SPF	"v=spf1 a mx a:mail.iesgn02.es ip4:146.59.196.83 ip6:2001:41d0:304:200::c9a1 ~all"
~~~

## Envío desde exterior a local

Hemos respondido al correo anterior:

![2](/assets/img/posts/correo/2.png)

Mostramos el log:

~~~
ID de mensaje	<CAFW_4e7YHemEKy35N12y+LwCTNwCoRCTFy4cs2sfHz3_MrcrbQ@mail.gmail.com>
Creado a las:	10 de febrero de 2021, 18:11 (entregado en 0 segundos)
De:	Alejandro Cabezas <alejandrocabezab@gmail.com>
Para:	Debian <debian@iesgn02.es>
Asunto:	Re: prueba desde ovh

MIME-Version: 1.0
Date: Wed, 10 Feb 2021 18:11:12 +0100
References: <20210210170805.B08B1A107C@ned.iesgn02.es>
In-Reply-To: <20210210170805.B08B1A107C@ned.iesgn02.es>
Message-ID: <CAFW_4e7YHemEKy35N12y+LwCTNwCoRCTFy4cs2sfHz3_MrcrbQ@mail.gmail.com>
Subject: Re: prueba desde ovh
From: Alejandro Cabezas <alejandrocabezab@gmail.com>
To: Debian <debian@iesgn02.es>
Content-Type: multipart/alternative; boundary="0000000000007088f505bafe7d73"

--0000000000007088f505bafe7d73
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: quoted-printable

Hola querido OVH

El mi=C3=A9, 10 feb 2021 a las 18:08, Debian (<debian@iesgn02.es>) escribi=
=C3=B3:

>
>

--=20
*Alejandro Cabezas*

--0000000000007088f505bafe7d73
Content-Type: text/html; charset="UTF-8"
Content-Transfer-Encoding: quoted-printable

<div dir=3D"ltr">Hola querido OVH</div><br><div class=3D"gmail_quote"><div =
dir=3D"ltr" class=3D"gmail_attr">El mi=C3=A9, 10 feb 2021 a las 18:08, Debi=
an (&lt;<a href=3D"mailto:debian@iesgn02.es">debian@iesgn02.es</a>&gt;) esc=
ribi=C3=B3:<br></div><blockquote class=3D"gmail_quote" style=3D"margin:0px =
0px 0px 0.8ex;border-left:1px solid rgb(204,204,204);padding-left:1ex"><br>
</blockquote></div><br clear=3D"all"><div><br></div>-- <br><div dir=3D"ltr"=
 class=3D"gmail_signature"><div dir=3D"ltr"><div><div dir=3D"ltr"><div dir=
=3D"ltr"><div><b><i>Alejandro Cabezas</i></b></div></div></div></div></div>=
</div>

--0000000000007088f505bafe7d73--
~~~

Para leerlo en el servidor OVH:

~~~
 mail
Mail version 8.1.2 01/15/2001.  Type ? for help.
"/var/mail/debian": 1 message 1 new
>N  1 alejandrocabezab@  Wed Feb 10 18:11   76/3627  Re: prueba desde ovh
& 
~~~

Y pulsamos el número 1:

~~~
References: <20210210170805.B08B1A107C@ned.iesgn02.es>
In-Reply-To: <20210210170805.B08B1A107C@ned.iesgn02.es>
From: Alejandro Cabezas <alejandrocabezab@gmail.com>
Date: Wed, 10 Feb 2021 18:11:12 +0100
Subject: Re: prueba desde ovh
To: Debian <debian@iesgn02.es>
Content-Type: multipart/alternative; boundary="00000000000014ce3d05bafe7ec6"

--00000000000014ce3d05bafe7ec6
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: quoted-printable

Hola querido OVH

El mi=C3=A9, 10 feb 2021 a las 18:08, Debian (<debian@iesgn02.es>) escribi=
=C3=B3:

>
>

--=20
*Alejandro Cabezas*

--00000000000014ce3d05bafe7ec6
Content-Type: text/html; charset="UTF-8"
Content-Transfer-Encoding: quoted-printable

<div dir=3D"ltr">Hola querido OVH</div><br><div class=3D"gmail_quote"><div =
dir=3D"ltr" class=3D"gmail_attr">El mi=C3=A9, 10 feb 2021 a las 18:08, Debi=
an (&lt;<a href=3D"mailto:debian@iesgn02.es">debian@iesgn02.es</a>&gt;) esc=
ribi=C3=B3:<br></div><blockquote class=3D"gmail_quote" style=3D"margin:0px =
0px 0px 0.8ex;border-left:1px solid rgb(204,204,204);padding-left:1ex"><br>
</blockquote></div><br clear=3D"all"><div><br></div>-- <br><div dir=3D"ltr"=
 class=3D"gmail_signature"><div dir=3D"ltr"><div><div dir=3D"ltr"><div dir=
=3D"ltr"><div><b><i>Alejandro Cabezas</i></b></div></div></div></div></div>=
</div>

--00000000000014ce3d05bafe7ec6--
~~~

Muestro el registro MX del servidor OVH:

~~~
dig MX iesgn02.es 

; <<>> DiG 9.11.5-P4-5.1+deb10u2-Debian <<>> MX iesgn02.es
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 29416
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;iesgn02.es.			IN	MX

;; ANSWER SECTION:
iesgn02.es.		3600	IN	MX	10 mail.iesgn02.es.

;; Query time: 33 msec
;; SERVER: 212.230.135.1#53(212.230.135.1)
;; WHEN: mié feb 10 18:16:46 CET 2021
;; MSG SIZE  rcvd: 60
~~~

## Envío de mail a usuario root, debian y correo exterior

Editamos el crontab desde el usuario debian:

~~~
crontab -e
~~~

Y añadimos lo siguiente:

~~~
MAILTO = root

55 18 * * * echo prueba crontab
~~~

Esto nos enviará un mail al usuario `root` cuando se realice la tarea:

~~~
root@ned:~# mail
Mail version 8.1.2 01/15/2001.  Type ? for help.
"/var/mail/root": 1 message 1 unread
>U  1 root@iesgn02.es    Wed Feb 10 18:55   23/709   Cron <debian@ned> echo prueba crontab
& 1
Message 1:
From debian@iesgn02.es  Wed Feb 10 18:55:01 2021
X-Original-To: root
From: root@iesgn02.es (Cron Daemon)
To: root@iesgn02.es
Subject: Cron <debian@ned> echo prueba crontab
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit
X-Cron-Env: <MAILTO=root>
X-Cron-Env: <SHELL=/bin/sh>
X-Cron-Env: <HOME=/home/debian>
X-Cron-Env: <PATH=/usr/bin:/bin>
X-Cron-Env: <LOGNAME=debian>
Date: Wed, 10 Feb 2021 18:55:01 +0100 (CET)

prueba crontab

& 
~~~

Para que se envíe al usuario debian, tendremos que crear un alias en:

~~~
sudo nano /etc/aliases
~~~

Añadimos lo siguiente:

~~~
postmaster:    root
root: debian
~~~

Y ejecutamos para actualizar el fichero:

~~~
sudo newaliases
~~~

Ahora nos llegará a nuestro usuario debian:

~~~
debian@ned:~$ mail
Mail version 8.1.2 01/15/2001.  Type ? for help.
"/var/mail/debian": 1 message 1 new
>N  1 root@iesgn02.es    Wed Feb 10 18:58   22/699   Cron <debian@ned> echo prueba crontab
& 1
Message 1:
From debian@iesgn02.es  Wed Feb 10 18:58:01 2021
X-Original-To: root
From: root@iesgn02.es (Cron Daemon)
To: root@iesgn02.es
Subject: Cron <debian@ned> echo prueba crontab
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit
X-Cron-Env: <MAILTO=root>
X-Cron-Env: <SHELL=/bin/sh>
X-Cron-Env: <HOME=/home/debian>
X-Cron-Env: <PATH=/usr/bin:/bin>
X-Cron-Env: <LOGNAME=debian>
Date: Wed, 10 Feb 2021 18:58:01 +0100 (CET)

prueba crontab

& 
~~~

Para que nos envíe el correo a un correo exterior tenemos que modificar el siguiente fichero:

~~~
sudo nano ~/.forward
~~~

Añadir lo siguiente:

~~~
alejandrocabezab@gmail.com
~~~

Y comprobamos en nuestro correo exterior:

![3](/assets/img/posts/correo/3.png)

## Configurar buzón Maildir

Para configurar un buzón maildir, nos vamos a:

~~~
sudo nano /etc/postfix/main.cf
~~~

Y añadimos/modificamos las siguientes líneas:

~~~
home_mailbox = Maildir/
mailbox_command =
~~~

Automáticamente cuando nos ha llegado el primer correo se ha creado la siguiente estructura:

~~~
tree Maildir/
Maildir/
├── cur
├── new
│   └── 1612984890.V801I419f6M898662.ned.iesgn02.es
└── tmp

3 directories, 1 file
~~~

Y para leerlo:

~~~
less Maildir/new/1612984890.V801I419f6M898662.ned.iesgn02.es

Return-Path: <alejandrocabezab@gmail.com>
X-Original-To: debian@iesgn02.es
Delivered-To: debian@iesgn02.es
Received: from mail-yb1-f179.google.com (mail-yb1-f179.google.com [209.85.219.179])
        by ned.iesgn02.es (Postfix) with ESMTPS id D8C00A12E3
        for <debian@iesgn02.es>; Wed, 10 Feb 2021 20:21:30 +0100 (CET)
Received: by mail-yb1-f179.google.com with SMTP id p186so3155672ybg.2
        for <debian@iesgn02.es>; Wed, 10 Feb 2021 11:21:30 -0800 (PST)
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        d=gmail.com; s=20161025;
        h=mime-version:from:date:message-id:subject:to;
        bh=A1m4L0hqThr1aDUumtZZUGt3FlwtKgJBxeOt2k328P4=;
        b=Suck4kWI0G8xVV/nNUB3M+uqiq6smRZ70jZz3RZry97Do0xnpOO9AC5uLfZX2eHmoU
         AuqOQvZmKOMS63/7IIk+DNNWtJkZYfQ+52uB+/uGRpRmrAn49D2+RRpmabiUFm4bxh25
         +w9K7cH3zTIuLJHHrccZW9wXREmuKkxqtRwrDmG98KBJy0fZKIdFeOHzQi3W/88XVnR3
         3ECKg1AK5jGKTmqXcyzL6RBBcs7Z5Om74fNGHtuQtEQd5RvpJASfgTNdZp86KmAT8FV0
         xiA87uFkYvR86RXbfkCE9RyvHxbfzEJk35Xw4DdypDwysiMH16djqcU6X4Hl8fLCTwto
         IqJQ==
X-Google-DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        d=1e100.net; s=20161025;
        h=x-gm-message-state:mime-version:from:date:message-id:subject:to;
        bh=A1m4L0hqThr1aDUumtZZUGt3FlwtKgJBxeOt2k328P4=;
        b=lQwz/yyJY+eeyzfYpFZbuPm1NxyzNTU4kM8ZsHycPd/xxzTWfi9yDsUCeqQN1mBrfx
         ucUlfErHA9eYiUW+ProX227Y9ltpsnVHVXrMY43LtOeOu/vuv+KQAGcTN9yEpt8D72z0
         BpRzuB2KcJGmlGLhVnQSeLQ8u1X98B6CzxEak/uBWAIGdooIBEPpjG533meLwnuZT+zM
         /TXNuv2hHi4TfDD/eHFsDrppb1IPlW3crMDkk0WdvfbMpLHjioe1wFJisH35VXkqYPh2
         v9qn6LiOK2H8o79YP13cSdFPQ0Wg12tVPscyFeoZcobEtURxpXS0+vLOztyWSpLmt1Lt
         lsNw==
X-Gm-Message-State: AOAM530v7IPlXqOkmA7oLIGIDedVM6hlTdx/mEdwTTFHCvP07KZKiICa
        lUSpOr+wxkqZ2MVxO5ZlV/CU0QSZpLkKUVbR7mUZIfbiG4Q=
X-Google-Smtp-Source: ABdhPJwg1+rT9mi2SplValrPoreQjnd9Y8t1vfSoDIJ9u25dake2BtfVISb5MNwDDYvSzaCuFOMCiOrAJat+ewQipBg=
X-Received: by 2002:a25:850f:: with SMTP id w15mr6511364ybk.487.1612984889802;
 Wed, 10 Feb 2021 11:21:29 -0800 (PST)
MIME-Version: 1.0
From: Alejandro Cabezas <alejandrocabezab@gmail.com>
Date: Wed, 10 Feb 2021 20:21:18 +0100
Message-ID: <CAFW_4e6sZSLsBfVnvFo_xmaVNwwrzwm09mC5WdjtGRik7Cs-wA@mail.gmail.com>
Subject: prueba maildir
To: Debian <debian@iesgn02.es>
Content-Type: multipart/alternative; boundary="00000000000062849e05bb004f8b"

--00000000000062849e05bb004f8b
Content-Type: text/plain; charset="UTF-8"

hola

-- 
*Alejandro Cabezas*

--00000000000062849e05bb004f8b
Content-Type: text/html; charset="UTF-8"
Content-Transfer-Encoding: quoted-printable

<div dir=3D"ltr">hola<br clear=3D"all"><div><br></div>-- <br><div dir=3D"lt=
r" class=3D"gmail_signature" data-smartmail=3D"gmail_signature"><div dir=3D=
"ltr"><div><div dir=3D"ltr"><div dir=3D"ltr"><div><b><i>Alejandro Cabezas</=
i></b></div></div></div></div></div></div></div>

--00000000000062849e05bb004f8b--
~~~

## Instalación y configuración protocolo IMAP para servicio externo

Hemos añadido en el DNS de OVH lo siguiente:

~~~
imap.iesgn02.es.
0	CNAME	ned
~~~

Hemos generado el certificado para el dominio _mail.iesgn02.es_, para ello:

~~~
sudo apt install certbot

sudo certbot --domains mail.iesgn02.es

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/ned.iesgn02.es/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/ned.iesgn02.es/privkey.pem
   Your cert will expire on 2021-05-11. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
~~~

Instalamos el paquete para poder ofrecer el protocolo _imap_:

~~~
apt-get install dovecot-imapd
~~~

Ahora nos vamos al fichero de configuracion:

~~~
sudo nano /etc/dovecot/conf.d/10-auth.conf
~~~

Y editamos las siguientes líneas para configurar el mecanismo de autenticación:

~~~
disable_plaintext_auth = yes
auth_username_format = %n
auth_mechanisms = plain login
~~~

También nos vamos a este otro fichero de configuración:

~~~
sudo nano /etc/dovecot/conf.d/10-mail.conf
~~~

Y cambiamos el _maildir_ para que _dovecot_ lo use:

~~~
#mail_location = mbox:~/mail:INBOX=/var/mail/%u
mail_location = maildir:~/Maildir
~~~

También tendremos que irnos al otro fichero de configuración de los certificados:

~~~
sudo nano /etc/dovecot/conf.d/10-ssl.conf
~~~

Y añadimos los certificados obtenidos por LetsEncrypt:

~~~
ssl = required

ssl_cert = </etc/letsencrypt/live/mail.iesgn02.es/fullchain.pem
ssl_key = </etc/letsencrypt/live/mail.iesgn02.es/privkey.pem

ssl_prefer_server_ciphers = yes

~~~

Y por último, editamos el fichero de configuración de dovecot

~~~
sudo nano /etc/dovecot/dovecot.conf
~~~

Y editamos las siguientes líneas para habilitar el protocolo:

~~~
protocols = imap
listen = *, ::
~~~

Habilitamos y reiniciamos _dovecot_ y _postfix_:

~~~
sudo systemctl enable dovecot
sudo systemctl restart dovecot
sudo systemctl restart postfix
~~~

Iniciamos Thunderbird y ponemos nuestros datos:

![4](/assets/img/posts/correo/4.png)

Comprobamos que recibimos los correos:

![5](/assets/img/posts/correo/5.png)

## Instalación y configuración SMTPS para envío externo

Vamos a configurar SMTPS, para ello, primero vamos a ir al fichero:

~~~
sudo nano /etc/postfix/master.cf
~~~

Y añadimos lo siguiente al final del fichero para habilitar el servicio de envío:

~~~
smtps     inet  n       -       y       -       -       smtpd
  -o syslog_name=postfix/smtps
  -o smtpd_tls_wrappermode=yes
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject
  -o smtpd_recipient_restrictions=permit_mynetworks,permit_sasl_authenticated,reject
  -o smtpd_sasl_type=dovecot
  -o smtpd_sasl_path=private/auth
~~~

También nos iremos al fichero:

~~~
sudo nano /etc/postfix/main.cf
~~~

Y añadiremos/cambiaremos las siguientes líneas para especificar la ubicación del certificado TLS(teniendo en cuenta que tiene que ser nuestro certificado creado anteriormente con cerbot):

~~~
# TLS parameters
smtpd_tls_cert_file=/etc/letsencrypt/live/mail.iesgn02.es/fullchain.pem
smtpd_tls_key_file=/etc/letsencrypt/live/mail.iesgn02.es/privkey.pem
smtpd_use_tls=yes
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache

#Enable TLS Encryption when Postfix sends outgoing emails
smtp_tls_security_level = may
smtp_tls_loglevel = 1
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache

#Enforce TLSv1.3 or TLSv1.2
smtpd_tls_mandatory_protocols = !SSLv2, !SSLv3, !TLSv1, !TLSv1.1
smtpd_tls_protocols = !SSLv2, !SSLv3, !TLSv1, !TLSv1.1
smtp_tls_mandatory_protocols = !SSLv2, !SSLv3, !TLSv1, !TLSv1.1
smtp_tls_protocols = !SSLv2, !SSLv3, !TLSv1, !TLSv1.1
~~~

Abrimos el fichero:

~~~
sudo nano /etc/dovecot/conf.d/10-master.conf
~~~

Y añadimos lo siguiente:

~~~
service auth {
  unix_listener /var/spool/postfix/private/auth {
      mode = 0660
      user = postfix
      group = postfix
  }
~~~

Reiniciamos ambos servicios:

~~~
sudo systemctl restart postfix
sudo systemctl restart dovecot
~~~

Hacemos una nueva configuración desde el cliente:

![6](/assets/img/posts/correo/6.png)

Envíamos un correo de prueba a gmail:

![7](/assets/img/posts/correo/7.png)

Vemos que ha llegado correctamente y cifrado:

![8](/assets/img/posts/correo/8.png)

## Instalación webmail roundcube

Vamos a configurar un webmail para nuestro servidor de correo, vamos a hacerlo con `docker`, el webmail es `roundcube`:

Lo primero que haremos será crear un nuevo virtualhost en nuestro nginx, para ello:

~~~
sudo nano /etc/nginx/sites-available/correo
~~~

Y le añadimos el siguiente contenido:

~~~
server {
  server_name correo.iesgn02.es;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header Host $host;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

  location / {
        proxy_pass http://127.0.0.1:8081;
  }
}
~~~

Lo habilitamos:

~~~
ln -s /etc/nginx/sites-available/correo /etc/nginx/sites-enabled/correo
~~~

Ya lo tendremos activo para montar nuestro docker en el puerto 8081, pero queremos que esté certificado, para ello hacemos uso de certbot:

~~~
sudo certbot --nginx
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx

Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: correo.iesgn02.es
2: mail.iesgn02.es
3: ned.iesgn02.es
4: portal.iesgn02.es
5: www.iesgn02.es
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel):
~~~

Seleccionamos `correo.iesgn02.es` y activamos la redirección.

Para crear nuestro docker ejecutamos la siguiente orden:

~~~
sudo docker run -e ROUNDCUBEMAIL_DEFAULT_HOST=tls://mail.iesgn02.es -e ROUNDCUBEMAIL_SMTP_SERVER=ssl://mail.iesgn02.es -e ROUNDCUBEMAIL_SMTP_PORT=465 -p 8081:80 -d roundcube/roundcubemail
~~~

Y comprobamos que podemos acceder y enviar correo correctamente:

![10](/assets/img/posts/correo/10.png)