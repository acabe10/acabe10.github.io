---
layout: post
title: Instalación y configuración inicial de OpenLDAP
tags: [all, OpenStack, Toboso, LDAP, OpenLDAP]
---
# Introducción

Buenas, en este post vamos realizar la instalación y configuración básica de OpenLDAP en nuestro escenario de OpenStack. Para realizar dicha instalación hemos modificado nuestro escenario de tal manera que ahora tenemos una cosa así:

![escenario2](/assets/img/posts/ldap/escenario2.png)

## Instalación de LDAP

Para la instalación de `ldap`:

~~~
sudo apt install slapd
~~~

Nos pedirá la contraseña del usuario `admin`, que es el usuario administrador de `ldap`. Si mostramos el árbol creado:

~~~
$ sudo slapcat

dn: dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: dcObject
objectClass: organization
o: cabezas.gonzalonazareno.org
dc: cabezas
structuralObjectClass: organization
entryUUID: 6dd36878-d1a4-103a-8d8b-9ba05eb682e8
creatorsName: cn=admin,dc=cabezas,dc=gonzalonazareno,dc=org
createTimestamp: 20201213153427Z
entryCSN: 20201213153427.163031Z#000000#000#000000
modifiersName: cn=admin,dc=cabezas,dc=gonzalonazareno,dc=org
modifyTimestamp: 20201213153427Z

dn: cn=admin,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: simpleSecurityObject
objectClass: organizationalRole
cn: admin
description: LDAP administrator
userPassword:: e1NTSEF9eEEwR3Iwc3RSL0o5RnBXTkNJRHJBLzUrNzhKbkpSUEE=
structuralObjectClass: organizationalRole
entryUUID: 6dd7d5de-d1a4-103a-8d8c-9ba05eb682e8
creatorsName: cn=admin,dc=cabezas,dc=gonzalonazareno,dc=org
createTimestamp: 20201213153427Z
entryCSN: 20201213153427.192112Z#000000#000#000000
modifiersName: cn=admin,dc=cabezas,dc=gonzalonazareno,dc=org
modifyTimestamp: 20201213153427Z
~~~

Como vemos, nos ha cogido como base (dn) el nombre que tenemos de dns. Nosotros queremos que el nombre base también contenga el nombre del equipo, para ello:

~~~
sudo dpkg-reconfigure -plow slapd
~~~

Y modificamos el nombre del equipo como también podemos modificar un par de parámetros más:

![1](/assets/img/posts/ldap/1.png)

![2](/assets/img/posts/ldap/2.png)

Si volvemos a ejecutar el comando anterior:

~~~
$ sudo slapcat

dn: dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: dcObject
objectClass: organization
o: Cabezas company
dc: freston
structuralObjectClass: organization
entryUUID: e7a48670-d1a7-103a-97b8-25b7f6f3554f
creatorsName: cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
createTimestamp: 20201213155920Z
entryCSN: 20201213155920.027962Z#000000#000#000000
modifiersName: cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
modifyTimestamp: 20201213155920Z

dn: cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: simpleSecurityObject
objectClass: organizationalRole
cn: admin
description: LDAP administrator
userPassword:: e1NTSEF9amhZcWJyRWs1T2FYcXZ2RVhDZkRpb0E2LzFuMS94Lzg=
structuralObjectClass: organizationalRole
entryUUID: e7aa82aa-d1a7-103a-97b9-25b7f6f3554f
creatorsName: cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
createTimestamp: 20201213155920Z
entryCSN: 20201213155920.067300Z#000000#000#000000
modifiersName: cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
modifyTimestamp: 20201213155920Z
~~~

También podemos confirmar el puerto en el que está escuchando el servidor ldap, el 389:

~~~
$ ss -lntp

State        Recv-Q        Send-Q               Local Address:Port               Peer Address:Port       
LISTEN       0             128                        0.0.0.0:22                      0.0.0.0:*          
LISTEN       0             128                        0.0.0.0:389                     0.0.0.0:*          
LISTEN       0             128                           [::]:22                         [::]:*          
LISTEN       0             128                           [::]:389                        [::]:*
~~~

Podemos también solicitar una visión general del directorio ldap con el siguiente comando, no sin antes instalar el paquete `ldap-utils`

~~~
$ ldapsearch -x -D "cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org" -b dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org -W

Enter LDAP Password: 
# extended LDIF
#
# LDAPv3
# base <dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# freston.cabezas.gonzalonazareno.org
dn: dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: dcObject
objectClass: organization
o: Cabezas company
dc: freston

# admin, freston.cabezas.gonzalonazareno.org
dn: cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: simpleSecurityObject
objectClass: organizationalRole
cn: admin
description: LDAP administrator
userPassword:: e1NTSEF9amhZcWJyRWs1T2FYcXZ2RVhDZkRpb0E2LzFuMS94Lzg=

# search result
search: 2
result: 0 Success

# numResponses: 3
# numEntries: 2
~~~

Si no queremos tener que introducir la base cada vez que solicitemos este tipo de peticiones la podemos cambiar editando el fichero:

~~~
sudo nano /etc/ldap/ldap.conf
~~~

Y añadiendo la siguiente línea:

~~~
BASE    dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
~~~

## Creación de unidades organizativas

Para poder crear las dos unidades organizativas vamos a crear un fichero con la extensión `ldif`:

~~~
nano uo.ldif
~~~

Con el siguiente contenido, el cuál creará dos unidades organizativas, una para las personas y otra para los grupos:

~~~
dn: ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
ou: People
objectClass: organizationalUnit

dn: ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
ou: Group
objectClass: organizationalUnit
~~~

Para añadirlo al directorio ldap:

~~~
ldapadd -x -D "cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org" -W -f uo.ldif

Enter LDAP Password: 
adding new entry "ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org"

adding new entry "ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org"
~~~

Y podemos confirmarlo ejecutándo el comando:

~~~
sudo slapcat
dn: dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: dcObject
objectClass: organization
o: Cabezas company
dc: freston
structuralObjectClass: organization
entryUUID: e7a48670-d1a7-103a-97b8-25b7f6f3554f
creatorsName: cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
createTimestamp: 20201213155920Z
entryCSN: 20201213155920.027962Z#000000#000#000000
modifiersName: cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
modifyTimestamp: 20201213155920Z

dn: cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: simpleSecurityObject
objectClass: organizationalRole
cn: admin
description: LDAP administrator
userPassword:: e1NTSEF9amhZcWJyRWs1T2FYcXZ2RVhDZkRpb0E2LzFuMS94Lzg=
structuralObjectClass: organizationalRole
entryUUID: e7aa82aa-d1a7-103a-97b9-25b7f6f3554f
creatorsName: cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
createTimestamp: 20201213155920Z
entryCSN: 20201213155920.067300Z#000000#000#000000
modifiersName: cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
modifyTimestamp: 20201213155920Z

dn: ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
ou: People
objectClass: organizationalUnit
structuralObjectClass: organizationalUnit
entryUUID: d8a37fb2-d1a9-103a-9dcb-cf27b271bd26
creatorsName: cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
createTimestamp: 20201213161313Z
entryCSN: 20201213161313.848953Z#000000#000#000000
modifiersName: cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
modifyTimestamp: 20201213161313Z

dn: ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
ou: Group
objectClass: organizationalUnit
structuralObjectClass: organizationalUnit
entryUUID: d8a7ebec-d1a9-103a-9dcc-cf27b271bd26
creatorsName: cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
createTimestamp: 20201213161313Z
entryCSN: 20201213161313.877974Z#000000#000#000000
modifiersName: cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
modifyTimestamp: 20201213161313Z
~~~