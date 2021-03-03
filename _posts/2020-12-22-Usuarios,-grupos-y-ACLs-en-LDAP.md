---
layout: post
title: Usuarios y grupos en LDAP
tags: [all, OpenStack, Toboso, LDAPs, OpenLDAP]
---
# Introducción

Buenas, en este post vamos a usar el servidor LDAP instalado [anteriormente](https://acabe10.github.io/2021-02-23-Instalacion-inicial-OpenLDAP/) para hacer varios ejercicios de añadir usuarios, grupos y ACLs.

# Ejercicios

## Crea 10 usuarios con los nombres que prefieras en LDAP, esos usuarios deben ser objetos de los tipos posixAccount e inetOrgPerson. Estos usuarios tendrán un atributo userPassword.

Para crear las contraseñas de los usuarios:

~~~
sudo slappasswd -v
New password: 
Re-enter new password: 
{SSHA}Y8cltEcp9dejk3s9MT9R+uNyxyU54nmR
~~~

Y creamos el fichero donde irán todos los usuarios:

~~~
nano users.ldif
~~~

Y añadimos lo siguiente:

~~~
dn: cn=grupo1,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: posixGroup
cn: grupo1
gidNumber: 2000

dn: uid=adrian,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: posixAccount
objectClass: inetOrgPerson
cn: Adrian Rodriguez Povea
sn: Rodriguez Povea
uid: adrian
uidNumber: 2001
gidNumber: 2000
homeDirectory: /home/adrian
loginShell: /bin/bash
userPassword: {SSHA}+WUt2/lu5IKTpD+rBMtcjbdJEtGY+n3e
mail: adrian@gmail.com
givenName: adrian

dn: uid=alejandro,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: posixAccount
objectClass: inetOrgPerson
cn: Alejandro Cabezas Barea
sn: Cabezas Barea
uid: alejandro
uidNumber: 2002
gidNumber: 2000
homeDirectory: /home/alejandro
loginShell: /bin/bash
userPassword: {SSHA}NfyfqlMR7l2ev0Kjpm7RVxYfaRQ+f7Z5
mail: alejandro@gmail.com
givenName: alejandro

dn: uid=celia,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: posixAccount
objectClass: inetOrgPerson
cn: Celia Garcia Marquez
sn: Garcia Marquez
uid: celia
uidNumber: 2003
gidNumber: 2000
homeDirectory: /home/celia
loginShell: /bin/bash
userPassword: {SSHA}iKNwrH3LIgJOLhTA/FG678Wzj1LJ+TLC
mail: celia@gmail.com
givenName: celia

dn: uid=guillermo,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: posixAccount
objectClass: inetOrgPerson
cn: Guillermo Vizcaino Rodriguez
sn: Vizcaino Rodriguez
uid: guillermo
uidNumber: 2004
gidNumber: 2000
homeDirectory: /home/guillermo
loginShell: /bin/bash
userPassword: {SSHA}PuC/X1ocHt1U4FK6KquBN575ZQW8cMI0
mail: guillermo@gmail.com
givenName: guillermo

dn: uid=ismael,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: posixAccount
objectClass: inetOrgPerson
cn: Ismael Santiago Estevez
sn: Santiago Estevez
uid: ismael
uidNumber: 2005
gidNumber: 2000
homeDirectory: /home/ismael
loginShell: /bin/bash
userPassword: {SSHA}8qIg+l21xQ4kCbHum7PRwch2I1BUeHx3
mail: ismael@gmail.com
givenName: ismael

dn: uid=javier,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: posixAccount
objectClass: inetOrgPerson
cn: Javier Crespillo Bergua
sn: Crespillo Bergua
uid: javier
uidNumber: 2006
gidNumber: 2000
homeDirectory: /home/javier
loginShell: /bin/bash
userPassword: {SSHA}xQb8e9Bh7tTzgmqWHw9s8mkJIS2BcpKY
mail: javier@gmail.com
givenName: javier

dn: uid=jonathan,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: posixAccount
objectClass: inetOrgPerson
cn: Jonathan Marquez Jimenez
sn: Marquez Jimenez
uid: jonathan
uidNumber: 2007
gidNumber: 2000
homeDirectory: /home/jonathan
loginShell: /bin/bash
userPassword: {SSHA}eSoft2bXJ5YVmJYpaZGMaTb+lG6y5okj
mail: jonathan@gmail.com
givenName: jonathan

dn: uid=jose,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: posixAccount
objectClass: inetOrgPerson
cn: Jose Calderon Frutos
sn: Calderon Frutos
uid: jose
uidNumber: 2008
gidNumber: 2000
homeDirectory: /home/jose
loginShell: /bin/bash
userPassword: {SSHA}b5BPSFnkBfw3UFWX2V1nrNLJ+0FE+kX+
mail: jose@gmail.com
givenName: jose

dn: uid=luis,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: posixAccount
objectClass: inetOrgPerson
cn: Luis Millan Hidalgo
sn: Millan Hidalgo
uid: luis
uidNumber: 2009
gidNumber: 2000
homeDirectory: /home/luis
loginShell: /bin/bash
userPassword: {SSHA}QjRavtx09NuP10uUp86TaRBYmNEZq2q0
mail: luis@gmail.com
givenName: luis

dn: uid=manuel,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: posixAccount
objectClass: inetOrgPerson
cn: Manuel Lora Roman
sn: Lora Roman
uid: manuel
uidNumber: 2010
gidNumber: 2000
homeDirectory: /home/manuel
loginShell: /bin/bash
userPassword: {SSHA}cYoMhxid9k/U35pqrm55r82EbudMKc7t
mail: manuel@gmail.com
givenName: manuel
~~~

Para añadirlos a ldap:

~~~
ldapadd -x -D "cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org" -W -f users.ldif
~~~

## Crea 3 grupos en LDAP dentro de una unidad organizativa diferente que sean objetos del tipo groupOfNames. Estos grupos serán: comercial, almacen y admin

Creamos el fichero:

~~~
nano groups.ldif
~~~

Y le añadimos el siguiente contenido:

~~~
dn: cn=comercial,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: groupOfNames
cn: comercial
member: 

dn: cn=almacen,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: groupOfNames
cn: almacen
member: 

dn: cn=admin,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: groupOfNames
cn: admin
member: 
~~~

Y añadimos a ldap:

~~~
ldapadd -x -D "cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org" -W -f groups.ldif
~~~

## Añade usuarios que pertenezcan a

Para poder modificar los miembros de un grupo, vamos a mostrar cada apartado del fichero que hemos creado a continuación, pero el fichero es un fichero único que importaremos al final.

### Solo al grupo comercial

~~~
dn: cn=comercial,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
changetype:modify
replace: member
member: uid=adrian,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
~~~

### Solo al grupo almacen

~~~
dn: cn=almacen,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
changetype:modify
replace: member
member: uid=ismael,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
~~~

### Al grupo comercial y almacen

~~~
dn: cn=comercial,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
changetype:modify
add: member
member: uid=celia,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org

dn: cn=almacen,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
changetype:modify
add: member
member: uid=celia,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
~~~

### Al grupo admin y comercial

~~~
dn: cn=admin,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
changetype:modify
replace: member
member: uid=guillermo,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org

dn: cn=comercial,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
changetype:modify
add: member
member: uid=guillermo,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
~~~

### Solo al grupo admin

~~~
dn: cn=admin,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
changetype:modify
add: member
member: uid=alejandro,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
~~~

### Importarlo a ldap

Creamos el fichero:

~~~
nano modify-groups.ldif
~~~

Añadimos todo lo anterior, y lo importamos a ldap:

~~~
ldapadd -x -D "cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org" -W -f modify-groups.ldif
~~~

## Modifica OpenLDAP apropiadamente para que se pueda obtener los grupos a los que pertenece cada usuario a través del atributo "memberOf".

### Módulo memberOf

Lo primero que tenemos que hacer es cargar el módulo que permite su consulta, creamos el fichero:

~~~
nano memberOf_config.ldif
~~~

Y le añadimos el siguiente contenido:

~~~
dn: cn=module,cn=config
cn: module
objectClass: olcModuleList
objectclass: top
olcModuleLoad: memberof.la
olcModulePath: /usr/lib/ldap

dn: olcOverlay={0}memberof,olcDatabase={1}mdb,cn=config
objectClass: olcConfig
objectClass: olcMemberOf
objectClass: olcOverlayConfig
objectClass: top
olcOverlay: memberof
olcMemberOfDangling: ignore
olcMemberOfRefInt: TRUE
olcMemberOfGroupOC: groupOfNames
olcMemberOfMemberAD: member
olcMemberOfMemberOfAD: memberOf
~~~

Cargamos el módulo:

~~~
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f memberOf_config.ldif 
~~~

### Integridad referencial

Para que los objetos tenga relación y no pierdan coherencia tenemos que cargar la integridad referencial, para ello:

~~~
nano int.ldif
~~~

Y añadimos el siguiente contenido:

~~~
dn: cn=module,cn=config
cn: module
objectclass: olcModuleList
objectclass: top
olcmoduleload: refint.la
olcmodulepath: /usr/lib/ldap

dn: olcOverlay={1}refint,olcDatabase={1}mdb,cn=config
objectClass: olcConfig
objectClass: olcOverlayConfig
objectClass: olcRefintConfig
objectClass: top
olcOverlay: {1}refint
olcRefintAttribute: memberof member manager owner

dn: olcOverlay=memberof,olcDatabase={1}mdb,cn=config
objectClass: olcOverlayConfig
objectClass: olcMemberOf
olcOverlay: memberof
olcMemberOfRefint: TRUE
~~~

Lo cargamos a ldap:

~~~
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f int.ldif
~~~

### Modificación grupos

Al haber cargado la configuración anterior después de crear los grupos, a dichos grupos no se le aplican las configuraciones anteriores, así que para eliminarlos:

~~~
sudo ldapdelete -x -D "cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org" 'cn=comercial,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org' -W
sudo ldapdelete -x -D "cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org" 'cn=almacen,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org' -W
sudo ldapdelete -x -D "cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org" 'cn=admin,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org' -W
~~~

Y los volvemos a añadir y a los usuarios:

~~~
ldapadd -x -D "cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org" -W -f groups.ldif
ldapadd -x -D "cn=admin,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org" -W -f modify-groups.ldif
~~~

### Consultas a memberOf

#### ¿De dónde es miembro uid=alejandro?

~~~
ldapsearch -LL -Y EXTERNAL -H ldapi:/// "(uid=alejandro)" -b dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org memberOf
SASL/EXTERNAL authentication started
SASL username: gidNumber=1000+uidNumber=1000,cn=peercred,cn=external,cn=auth
SASL SSF: 0
version: 1

dn: uid=alejandro,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
memberOf: cn=admin,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
~~~

#### ¿De dónde es miembro uid=guillermo?

~~~
ldapsearch -LL -Y EXTERNAL -H ldapi:/// "(uid=guillermo)" -b dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org memberOf
SASL/EXTERNAL authentication started
SASL username: gidNumber=1000+uidNumber=1000,cn=peercred,cn=external,cn=auth
SASL SSF: 0
version: 1

dn: uid=guillermo,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
memberOf: cn=admin,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
memberOf: cn=comercial,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
~~~

#### ¿De dónde es miembro uid=celia?

~~~
ldapsearch -LL -Y EXTERNAL -H ldapi:/// "(uid=celia)" -b dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org memberOf
SASL/EXTERNAL authentication started
SASL username: gidNumber=1000+uidNumber=1000,cn=peercred,cn=external,cn=auth
SASL SSF: 0
version: 1

dn: uid=celia,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
memberOf: cn=comercial,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
memberOf: cn=almacen,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
~~~

#### ¿De dónde es miembro uid=ismael?

~~~
ldapsearch -LL -Y EXTERNAL -H ldapi:/// "(uid=ismael)" -b dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org memberOf
SASL/EXTERNAL authentication started
SASL username: gidNumber=1000+uidNumber=1000,cn=peercred,cn=external,cn=auth
SASL SSF: 0
version: 1

dn: uid=ismael,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
memberOf: cn=almacen,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
~~~

#### ¿De dónde es miembro uid=adrian?

~~~
ldapsearch -LL -Y EXTERNAL -H ldapi:/// "(uid=adrian)" -b dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org memberOf
SASL/EXTERNAL authentication started
SASL username: gidNumber=1000+uidNumber=1000,cn=peercred,cn=external,cn=auth
SASL SSF: 0
version: 1

dn: uid=adrian,ou=People,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
memberOf: cn=comercial,ou=Group,dc=freston,dc=cabezas,dc=gonzalonazareno,dc=org
~~~