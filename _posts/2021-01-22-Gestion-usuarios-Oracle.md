---
layout: post
title: Gestión de usuarios Oracle
tags: [all, Oracle, Debian, BBDD]
---
# Introducción

Buenas, en este post vamos a hacer una serie de ejercicios relacionados con la gestión de usuarios en OracleDB.

#### Crea un usuario llamado Becario y, sin usar los roles de ORACLE, dale los siguientes privilegios:

~~~
CREATE USER becario
IDENTIFIED BY becario;
~~~

* Conectarse a la base de datos.

	~~~
	GRANT CREATE SESSION TO becario;
	~~~

* Modificar el número de errores en la introducción de la contraseña de cualquier usuario.
	
	Vamos a usar perfiles en Oracle, para ello, primero los activamos:
	~~~
	ALTER SYSTEM SET RESOURCE_LIMIT=TRUE;
	~~~

	Le damos el privilegio a becario para que pueda crear perfiles:
	~~~
	GRANT CREATE PROFILE, ALTER PROFILE, DROP PROFILE TO becario;
	~~~

	Creamos uno que se llamará ERRORPASSWORD:
	~~~
	CREATE PROFILE errorpassword LIMIT;
	~~~

	Y modificamos dicho perfil para que el número de errores sea el deseado:

	~~~
	ALTER PROFILE errorpassword LIMIT
		FAILED_LOGIN_ATTEMPTS 5;
	~~~

	También le damos el privilegio a becario para que pueda modificar usuarios:
	~~~
	GRANT ALTER USER TO becario;
	~~~

	Y asignamos el perfil al usuario deseado:
	~~~
	ALTER USER becario PROFILE errorpassword;
	~~~

* Modificar índices en cualquier esquema (este privilegio podrá pasarlo a quien quiera)
	
	~~~
	GRANT INDEX ON *.* TO becario WITH GRANT OPTION;
	~~~

* Insertar filas en scott.emp (este privilegio podrá pasarlo a quien quiera)
	
	~~~
	GRANT INSERT ON scott.emp TO becario WITH GRANT OPTION;
	~~~

* Crear objetos en cualquier tablespace.
	
	~~~
	GRANT CREATE ANY TABLE TO becario;
	~~~

* Gestión completa de usuarios, privilegios y roles.

	~~~
	GRANT ALL PRIVILEGES TO becario;
	~~~

#### Realiza una función de verificación de contraseñas que compruebe que la contraseña difiere en más de cinco caracteres de la anterior y que la longitud de la misma es diferente de la anterior. Asígnala al perfil CONTRASEÑASEGURA. Comprueba que funciona correctamente.

Creamos la función:

~~~
CREATE OR REPLACE FUNCTION password_verify
(
old_password VARCHAR2,
new_password VARCHAR2
)
RETURN boolean
AS
v_mayus_old VARCHAR2(30):=UPPER(old_password);
v_mayus_new VARCHAR2(30):=UPPER(new_password);
v_dif NUMBER:=0;
v_valida NUMBER:=0;

BEGIN
FOR i IN 1..LENGTH(v_mayus_new) LOOP
IF INSTR(v_mayus_old,SUBSTR(v_mayus_new,i,1),1) = 0
THEN
v_dif:=v_dif+1;
END IF;
END LOOP;
IF v_dif < 5 THEN
RAISE_APPLICATION_ERROR(-20001,'Debe cambiar en mas
de 5 caracteres');
END IF;
IF LENGTH(old_password) = LENGTH(new_password) THEN
RAISE_APPLICATION_ERROR(-20002,'Debe ser de
diferente longitud');
END IF;
RETURN TRUE;
END password_verify;
/
~~~

Activamos los perfiles en Oracle:

~~~
ALTER SYSTEM SET RESOURCE_LIMIT=TRUE
~~~

Creamos el perfil y le asignamos la función:

~~~
CREATE PROFILE passwordsegura LIMIT
PASSWORD_VERIFY_FUNCTION password_verify;
~~~

#### Realiza un procedimiento llamado MostrarPrivilegiosdelRol que reciba el nombre de un rol y muestre los privilegios de sistema y los privilegios sobre objetos que lo componen.

~~~
CREATE OR REPLACE PROCEDURE MostrarPrivilegiosDelRol
(
p_nombre_rol role_tab_privs.role%TYPE
)
AS
	CURSOR c_priv_sys IS
	SELECT PRIVILEGE
	FROM role_sys_privs
	WHERE role=p_nombre_rol;

	CURSOR c_priv_tab IS
	SELECT OWNER,TABLE_NAME,PRIVILEGE
	FROM role_tab_privs
	WHERE role=p_nombre_rol;

BEGIN

	ComprobarRolInexistente(p_nombre_rol);
	
	DBMS_OUTPUT.PUT_LINE(chr(9));
	DBMS_OUTPUT.PUT_LINE('Nombre rol: '||p_nombre_rol);
	DBMS_OUTPUT.PUT_LINE(chr(9));
	DBMS_OUTPUT.PUT_LINE('Privilegios de sistema:');
	
	FOR v_privs_sys IN c_priv_sys LOOP
		DBMS_OUTPUT.PUT_LINE(chr(9));
		DBMS_OUTPUT.PUT_LINE(v_privs_sys.PRIVILEGE);
	END LOOP;

	DBMS_OUTPUT.PUT_LINE(chr(9));
	DBMS_OUTPUT.PUT_LINE('Privilegios sobre objetos:');

	FOR v_privs_tab IN c_priv_tab LOOP
		DBMS_OUTPUT.PUT_LINE(chr(9));
		DBMS_OUTPUT.PUT_LINE('Propietario: '||
		v_privs_tab.OWNER);
		DBMS_OUTPUT.PUT_LINE('Nombre tabla: '||
		v_privs_tab.TABLE_NAME);
		DBMS_OUTPUT.PUT_LINE('Privilegio: '||
		v_privs_tab.PRIVILEGE);
	END LOOP;

END MostrarPrivilegiosDelRol;
/
~~~

~~~
CREATE OR REPLACE PROCEDURE ComprobarRolInexistente
(
	p_nombre_rol role_tab_privs.role%TYPE
)
AS
	v_count_rol NUMBER;
BEGIN
	SELECT COUNT(*) INTO v_count_rol
	FROM DBA_ROLES
	WHERE ROLE=p_nombre_rol;

	IF v_count_rol=0 THEN
		RAISE_APPLICATION_ERROR(-20104,'Rol no existe');
	END IF;
END ComprobarRolInexistente;
/
~~~

Resultado:

~~~
SQL> exec MostrarPrivilegiosDelRol('PRUEBA');
	Nombre rol: PRUEBA
	Privilegios de sistema:
	CREATE USER
	Privilegios sobre objetos:
	Propietario: PRUEBA2
	Nombre tabla: POBLACIONES
	Privilegio: SELECT
	Propietario: SYS
	Nombre tabla: PROVINCIAS
	Privilegio: SELECT
PL/SQL procedure successfully completed.
~~~

#### Realiza un procedimiento llamado PermisosdeAsobreB que reciba dos nombres de usuario y muestre los permisos que tiene el primero de ellos sobre objetos del segundo.

~~~
CREATE OR REPLACE PROCEDURE ComprobarRolInexistente
(
	p_nombre_rol role_tab_privs.role%TYPE
)
AS
	v_count_rol NUMBER;
BEGIN
	SELECT COUNT(*) INTO v_count_rol
	FROM DBA_ROLES
	WHERE ROLE=p_nombre_rol;

	IF v_count_rol=0 THEN
		RAISE_APPLICATION_ERROR(-20104,'Rol no existe');
	END IF;
END ComprobarRolInexistente;
/
~~~

~~~
CREATE OR REPLACE PROCEDURE ComprobarUsuInexistente
(
	p_usuario dba_tab_privs.OWNER%TYPE
)
AS
	v_count_usu NUMBER;
BEGIN

	SELECT COUNT(*) INTO v_count_usu
	FROM DBA_USERS
	WHERE USERNAME=p_usuario;

	IF v_count_usu=0 THEN
		RAISE_APPLICATION_ERROR(-20105,'Usuario no existe');
	END IF;
END ComprobarUsuInexistente;
/
~~~

Resultado:

~~~
SQL> exec PermisosDeAsobreB('PRUEBA1','PRUEBA2');
	Permisos sobre objetos de PRUEBA1 sobre PRUEBA2
	Propietario: PRUEBA2
	Nombre tabla: POBLACIONES
	Privilegio: SELECT
	Privilegiado: PRUEBA1
PL/SQL procedure successfully completed.
~~~

#### Realiza un procedimiento llamado MostrarInfoPerfil que reciba el nombre de un perfil y muestre su composición y los usuarios que lo tienen asignado.

Creamos un perfil de prueba:

~~~
CREATE PROFILE lim_prueba LIMIT CONNECT_TIME 45;
Procedimientos:
~~~

~~~
CREATE OR REPLACE PROCEDURE MostrarInfoPerfil
(
	p_perfil dba_profiles.profile%TYPE
)
AS
	CURSOR c_perfiles IS
	SELECT RESOURCE_NAME,LIMIT
	FROM dba_profiles
	WHERE profile=p_perfil;

	CURSOR c_usuarios_asignados IS
	SELECT USERNAME
	FROM dba_users
	WHERE PROFILE=p_perfil;
BEGIN
	ComprobarPerfilInexistente(p_perfil);

	DBMS_OUTPUT.PUT_LINE(chr(9));
	DBMS_OUTPUT.PUT_LINE('Composicion perfil '|| p_perfil||':');

	FOR v_perfil IN c_perfiles LOOP
		DBMS_OUTPUT.PUT_LINE(chr(9));
		DBMS_OUTPUT.PUT_LINE('Atributo: '|| v_perfil.RESOURCE_NAME);
		DBMS_OUTPUT.PUT_LINE('Valor: '||v_perfil.LIMIT);
	END LOOP;

	DBMS_OUTPUT.PUT_LINE(chr(9));
	DBMS_OUTPUT.PUT_LINE(chr(9));
	DBMS_OUTPUT.PUT_LINE('Usuarios que tienen el perfil '|| p_perfil||':');

	FOR v_usuarios IN c_usuarios_asignados LOOP
		DBMS_OUTPUT.PUT_LINE(chr(9));
		DBMS_OUTPUT.PUT_LINE('Nombre usuario: '|| v_usuarios.USERNAME);
	END LOOP;
END MostrarInfoPerfil;
/
~~~

~~~
CREATE OR REPLACE PROCEDURE ComprobarPerfilInexistente
(
	p_perfil dba_profiles.profile%TYPE
)
AS
	v_count_perfil NUMBER;
BEGIN
	SELECT COUNT(*) INTO v_count_perfil
	FROM DBA_PROFILES
	WHERE PROFILE=p_perfil;

	IF v_count_perfil=0 THEN
		RAISE_APPLICATION_ERROR(-20106,'Perfil no existe');
	END IF;
END ComprobarPerfilInexistente;
/
~~~

Resultado:

~~~
SQL> exec MostrarInfoPerfil('LIM_PRUEBA');
	Composicion perfil LIM_PRUEBA:
	Atributo: COMPOSITE_LIMIT
	Valor: DEFAULT
	Atributo: SESSIONS_PER_USER
	Valor: DEFAULT
	Atributo: CPU_PER_SESSION
	Valor: DEFAULT
	Atributo: CPU_PER_CALL
	Valor: DEFAULT
	Atributo: LOGICAL_READS_PER_SESSION
	Valor: DEFAULT
	Atributo: LOGICAL_READS_PER_CALL
	Valor: DEFAULT
	Atributo: IDLE_TIME
	Valor: DEFAULT
	Atributo: CONNECT_TIME
	Valor: 45
	Atributo: PRIVATE_SGA
	Valor: DEFAULT
	Atributo: FAILED_LOGIN_ATTEMPTS
	Valor: DEFAULT
	Atributo: PASSWORD_LIFE_TIME
	Valor: DEFAULT
	Atributo: PASSWORD_REUSE_TIME
	Valor: DEFAULT
	Atributo: PASSWORD_REUSE_MAX
	Valor: DEFAULT
	Atributo: PASSWORD_VERIFY_FUNCTION
	Valor: DEFAULT
	Atributo: PASSWORD_LOCK_TIME
	Valor: DEFAULT
	Atributo: PASSWORD_GRACE_TIME
	Valor: DEFAULT
	Usuarios que tienen el perfil LIM_PRUEBA:
	Nombre usuario: PRUEBA1
PL/SQL procedure successfully completed.
~~~

#### (ORACLE, Postgres) Realiza un procedimiento que reciba un nombre de usuario y nos muestre cuántas sesiones tiene abiertas en este momento. Además, para cada una de dichas sesiones nos mostrará la hora de comienzo y el nombre de la máquina, sistema operativo y programa desde el que fue abierta.

~~~
CREATE OR REPLACE PROCEDURE MostrarSesionesAbiertas
(
	p_usuario V$SESSION.USERNAME%TYPE
)
AS
	CURSOR c_sesiones IS
	SELECT LOGON_TIME,MACHINE,PROGRAM
	FROM V$SESSION
	WHERE USERNAME=p_usuario
	AND STATUS='ACTIVE';
BEGIN
	ComprobarUsuInexistente(p_usuario);
	ComprobarSesionesInactivas(p_usuario);

	DBMS_OUTPUT.PUT_LINE(chr(9));
	DBMS_OUTPUT.PUT_LINE('Sesiones abiertas de '|| p_usuario||':');

	FOR v_sesion IN c_sesiones LOOP
		DBMS_OUTPUT.PUT_LINE(chr(9));
		DBMS_OUTPUT.PUT_LINE('Login: '|| v_sesion.LOGON_TIME);
		DBMS_OUTPUT.PUT_LINE('Nombre host: '|| v_sesion.MACHINE);
		DBMS_OUTPUT.PUT_LINE('Programa: '|| v_sesion.PROGRAM);
	END LOOP;
 END MostrarSesionesAbiertas;
/
~~~

~~~
CREATE OR REPLACE PROCEDURE ComprobarSesionesInactivas
(
	p_usuario V$SESSION.USERNAME%TYPE
)
AS
	v_count_sesion NUMBER;
BEGIN
	SELECT COUNT(*) INTO v_count_sesion
	FROM V$SESSION
	WHERE USERNAME=p_usuario
	AND STATUS='ACTIVE';

	IF v_count_sesion=0 THEN
		RAISE_APPLICATION_ERROR(-20107,'El usuario '|| p_usuario||' no tiene sesiones activas');
	END IF;
END ComprobarSesionesInactivas;
/
~~~

Resultado:

~~~
SQL> exec MostrarSesionesAbiertas('SYS');
	Sesiones abiertas de SYS:
	Login: 01-FEB-21
	Nombre host: oracle
	Programa: sqlplus@oracle (TNS V1-V3)
PL/SQL procedure successfully completed.
~~~

#### Realiza un procedimiento que muestre los usuarios que pueden conceder privilegios de sistema a otros usuarios y cuales son dichos privilegios.

~~~
CREATE OR REPLACE PROCEDURE MostrarUsuariosPrivilegios
AS
	CURSOR c_privi IS
	SELECT GRANTEE,PRIVILEGE
	FROM dba_sys_privs
	WHERE ADMIN_OPTION='YES'
	GROUP BY GRANTEE,PRIVILEGE
	ORDER BY GRANTEE;
BEGIN
	FOR v_privs IN c_privi LOOP
		DBMS_OUTPUT.PUT_LINE(chr(9));
		DBMS_OUTPUT.PUT_LINE('Usuario: '||v_privs.GRANTEE);
		DBMS_OUTPUT.PUT_LINE('Privilegio que puede conceder: '||v_privs.PRIVILEGE);
	END LOOP;
END MostrarUsuariosPrivilegios;
/
~~~

Resultado:

~~~
SQL> exec MostrarUsuariosPrivilegios;
	Usuario: APEX_040200
	Privilegio que puede conceder: CREATE CLUSTER
	Usuario: APEX_040200
	Privilegio que puede conceder: CREATE DIMENSION
	Usuario: APEX_040200
	Privilegio que puede conceder: CREATE INDEXTYPE
	Usuario: APEX_040200
	Privilegio que puede conceder: CREATE JOB
	Usuario: APEX_040200
	Privilegio que puede conceder: CREATE MATERIALIZED VIEW
	Usuario: APEX_040200
	Privilegio que puede conceder: CREATE OPERATOR
	Usuario: APEX_040200
	Privilegio que puede conceder: CREATE PROCEDURE
	Usuario: APEX_040200
	Privilegio que puede conceder: CREATE SEQUENCE
	Usuario: APEX_040200
	Privilegio que puede conceder: CREATE SESSION
	Usuario: APEX_040200
	Privilegio que puede conceder: CREATE SYNONYM
	Usuario: APEX_040200
	Privilegio que puede conceder: CREATE TABLE
	Usuario: APEX_040200
	Privilegio que puede conceder: CREATE TRIGGER
	Usuario: APEX_040200
	Privilegio que puede conceder: CREATE TYPE
	Usuario: APEX_040200
	Privilegio que puede conceder: CREATE VIEW
	Usuario: AQ_ADMINISTRATOR_ROLE
	Privilegio que puede conceder: CREATE EVALUATION CONTEXT
	Usuario: AQ_ADMINISTRATOR_ROLE
	Privilegio que puede conceder: CREATE RULE
	Usuario: AQ_ADMINISTRATOR_ROLE
	Privilegio que puede conceder: CREATE RULE SET
	Usuario: AQ_ADMINISTRATOR_ROLE
	Privilegio que puede conceder: DEQUEUE ANY QUEUE
	Usuario: AQ_ADMINISTRATOR_ROLE
	Privilegio que puede conceder: ENQUEUE ANY QUEUE
	Usuario: AQ_ADMINISTRATOR_ROLE
	Privilegio que puede conceder: MANAGE ANY QUEUE
	Usuario: DVSYS
	Privilegio que puede conceder: CREATE EVALUATION CONTEXT
	Usuario: DVSYS
	Privilegio que puede conceder: CREATE RULE
	Usuario: DVSYS
	Privilegio que puede conceder: CREATE RULE SET
	Usuario: DV_ACCTMGR
	Privilegio que puede conceder: CREATE SESSION
	Usuario: PRUEBA1
	Privilegio que puede conceder: CREATE USER
	Usuario: SCHEDULER_ADMIN
	Privilegio que puede conceder: CREATE ANY CREDENTIAL
	Usuario: SCHEDULER_ADMIN
	Privilegio que puede conceder: CREATE ANY JOB
	Usuario: SCHEDULER_ADMIN
	Privilegio que puede conceder: CREATE CREDENTIAL
	Usuario: SCHEDULER_ADMIN
	Privilegio que puede conceder: CREATE EXTERNAL JOB
	Usuario: SCHEDULER_ADMIN
	Privilegio que puede conceder: CREATE JOB
	Usuario: SCHEDULER_ADMIN
	Privilegio que puede conceder: EXECUTE ANY CLASS
	Usuario: SCHEDULER_ADMIN
	Privilegio que puede conceder: EXECUTE ANY PROGRAM
	Usuario: SCHEDULER_ADMIN
	Privilegio que puede conceder: MANAGE SCHEDULER
	Usuario: SYS
	Privilegio que puede conceder: ALTER ANY EVALUATION CONTEXT
	Usuario: SYS
	Privilegio que puede conceder: ALTER ANY RULE
	Usuario: SYS
	Privilegio que puede conceder: ALTER ANY RULE SET
	Usuario: SYS
	Privilegio que puede conceder: CREATE ANY EVALUATION CONTEXT
	Usuario: SYS
	Privilegio que puede conceder: CREATE ANY RULE
	Usuario: SYS
	Privilegio que puede conceder: CREATE ANY RULE SET
	Usuario: SYS
	Privilegio que puede conceder: CREATE EVALUATION CONTEXT
	Usuario: SYS
	Privilegio que puede conceder: CREATE RULE
	Usuario: SYS
	Privilegio que puede conceder: CREATE RULE SET
	Usuario: SYS
	Privilegio que puede conceder: DELETE ANY TABLE
	Usuario: SYS
	Privilegio que puede conceder: DEQUEUE ANY QUEUE
	Usuario: SYS
	Privilegio que puede conceder: DROP ANY EVALUATION CONTEXT
	Usuario: SYS
	Privilegio que puede conceder: DROP ANY RULE
	Usuario: SYS
	Privilegio que puede conceder: DROP ANY RULE SET
	Usuario: SYS
	Privilegio que puede conceder: ENQUEUE ANY QUEUE
	Usuario: SYS
	Privilegio que puede conceder: EXECUTE ANY EVALUATION CONTEXT
	Usuario: SYS
	Privilegio que puede conceder: EXECUTE ANY RULE
	Usuario: SYS
	Privilegio que puede conceder: EXECUTE ANY RULE SET
	Usuario: SYS
	Privilegio que puede conceder: INSERT ANY TABLE
	Usuario: SYS
	Privilegio que puede conceder: MANAGE ANY QUEUE
	Usuario: SYS
	Privilegio que puede conceder: READ ANY TABLE
	Usuario: SYS
	Privilegio que puede conceder: SELECT ANY TABLE
	Usuario: SYS
	Privilegio que puede conceder: UPDATE ANY TABLE
	Usuario: SYSKM
	Privilegio que puede conceder: ADMINISTER KEY MANAGEMENT
	Usuario: SYSTEM
	Privilegio que puede conceder: DEQUEUE ANY QUEUE
	Usuario: SYSTEM
	Privilegio que puede conceder: ENQUEUE ANY QUEUE
	Usuario: SYSTEM
	Privilegio que puede conceder: MANAGE ANY QUEUE
PL/SQL procedure successfully completed.
~~~