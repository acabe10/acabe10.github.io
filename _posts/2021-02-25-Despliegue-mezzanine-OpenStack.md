---
layout: post
title: Despliegue Mezzanine, aplicación web python en OpenStack
tags: [all, OpenStack, Toboso, Servidor web, Python, Mezzanine]
---
# Introducción

Buenas, en este post vamos a desplegar Mezzanine (CMS Python) en nuestro escenario de OpenStack.

Primero lo haremos en una máquina en desarrollo y luego lo pasaremos a producción.

## Instalación en desarrollo

Vamos a usar Mezzanine, para ello primero vamos a crear un entorno virtual en desarrollo y lo activamos:

~~~
python3 -m venv ~/entornos/mezzanine
source ~/entornos/mezzanine/bin/activate
~~~

Instamos mezzanine y un paquete necesario:

~~~
pip install wheel
pip install mezzanine
~~~

Activamos el sitio de mezzanine:

~~~
cd ~/entornos/mezzanine
mezzanine-project mysite
~~~

Creamos la base de datos necesaria:

~~~
cd mysite
python3 manage.py migrate
~~~

Creamos el superusuario:

~~~
python3 manage.py createsuperuser
~~~

Y ejecutamos la aplicación:

~~~
python manage.py runserver
              .....
          _d^^^^^^^^^b_
       .d''           ``b.
     .p'                `q.
    .d'                   `b.
   .d'                     `b.   * Mezzanine 4.3.1
   ::                       ::   * Django 1.11.29
  ::    M E Z Z A N I N E    ::  * Python 3.7.3
   ::                       ::   * SQLite 3.27.2
   `p.                     .q'   * Linux 4.19.0-13-amd64
    `p.                   .q'
     `b.                 .d'
       `q..          ..p'
          ^q........p^
              ''''

Performing system checks...

System check identified no issues (0 silenced).
January 27, 2021 - 16:51:17
Django version 1.11.29, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
~~~

Accedemos a la aplicación:

![1](/assets/img/posts/mezzanine/1.png)

### Personalización página

Primero vamos a cambiar el nombre, para ello accedemos al panel de administración introduciendo el usuario anteriormente creado:

![2](/assets/img/posts/mezzanine/2.png)

Nos vamos a `settings` y cambiamos el apartado llamado `Site Title`:

![3](/assets/img/posts/mezzanine/3.png)

También hemos añadido una página nueva entrando en el apartado `Pages` y creando una nueva:

![6](/assets/img/posts/mezzanine/6.png)

Y nuestra página ha quedado de la siguiente forma:

![5](/assets/img/posts/mezzanine/5.png)

### Backup base de datos

Vamos a hacer una copia de la base de datos del proyecto:

~~~
python manage.py dumpdata > backup_db.json
~~~

### Subimos a github

Creado anteriormente un repositorio en GitHub, vamos a subir los ficheros generados:

~~~
git add .
git commit -am "subo a git"
git push
~~~

## Instalación en producción

Vamos a instalar `Mezzanine` en `quijote`, ya que tenemos el servidor web alojado ahí.

### Git y git clone

Para tener los datos en `quijote` debemos de instalar git para poder hacer una clonación del repositorio:

~~~
sudo dnf install git
cd /var/www
sudo git clone https://github.com/acabe10/mezzanine.git
~~~

### Entorno virtual y dependencias

Vamos a crear un entorno virtual para usar la aplicación:

~~~
mkdir ~/entornos
python3 -m venv ~/entornos/mezzanine
~~~

Lo activamos e instalamos las dependencias:

~~~
source ~/entornos/mezzanine/bin/activate
pip install -r mezzanine/mysite/requirements.txt
~~~

### Apache2 y módulo wsgi

Para que se pueda ejecutar código python en nuestro apache2 tendremos que instalar el módulo de centos que permite ejecutarlo:

~~~
sudo dnf install python3-mod_wsgi
~~~

También tendremos que instalar el módulo wsgi en el entorno virtual, no sin antes instalar en nuestra máquina lo siguiente:

~~~
sudo dnf install gcc python3-devel
pip install uwsgi
~~~

Podemos probar uwsgi de la siguiente forma:

~~~
sudo ~/entornos/mezzanine/bin/uwsgi --http 8080 --plugin python3 --chdir /var/www/mezzanine/mysite/ --wsgi-file /var/www//mezzanine/mysite/wsgi.py --process 4 --threads 2 --master
open("./python3_plugin.so"): No such file or directory [core/utils.c line 3732]
!!! UNABLE to load uWSGI plugin: ./python3_plugin.so: cannot open shared object file: No such file or directory !!!
*** Starting uWSGI 2.0.19.1 (64bit) on [Thu Jan 28 12:37:44 2021] ***
compiled with version: 8.3.1 20191121 (Red Hat 8.3.1-5) on 28 January 2021 10:54:52
os: Linux-4.18.0-240.1.1.el8_3.x86_64 #1 SMP Thu Nov 19 17:20:08 UTC 2020
nodename: quijote
machine: x86_64
clock source: unix
detected number of CPU cores: 1
current working directory: /var/www
detected binary path: /home/centos/entornos/mezzanine/bin/uwsgi
!!! no internal routing support, rebuild with pcre support !!!
uWSGI running as root, you can use --uid/--gid/--chroot options
*** WARNING: you are running uWSGI as root !!! (use the --uid flag) *** 
chdir() to /var/www/mezzanine/mysite/
your processes number limit is 1649
your memory page size is 4096 bytes
detected max file descriptor number: 1024
lock engine: pthread robust mutexes
thunder lock: disabled (you can enable it with --thunder-lock)
uWSGI http bound on 8080 fd 4
uwsgi socket 0 bound to TCP address 127.0.0.1:36639 (port auto-assigned) fd 3
uWSGI running as root, you can use --uid/--gid/--chroot options
*** WARNING: you are running uWSGI as root !!! (use the --uid flag) *** 
Python version: 3.6.8 (default, Aug 24 2020, 17:57:11)  [GCC 8.3.1 20191121 (Red Hat 8.3.1-5)]
Python main interpreter initialized at 0x8e66c0
uWSGI running as root, you can use --uid/--gid/--chroot options
*** WARNING: you are running uWSGI as root !!! (use the --uid flag) *** 
python threads support enabled
your server socket listen backlog is limited to 100 connections
your mercy for graceful operations on workers is 60 seconds
mapped 416720 bytes (406 KB) for 8 cores
*** Operational MODE: preforking+threaded ***
failed to open python file /var/www//mezzanine/mysite/wsgi.py
unable to load app 0 (mountpoint='') (callable not found or import error)
*** no app loaded. going in full dynamic mode ***
uWSGI running as root, you can use --uid/--gid/--chroot options
*** WARNING: you are running uWSGI as root !!! (use the --uid flag) *** 
*** uWSGI is running in multiple interpreter mode ***
spawned uWSGI master process (pid: 105733)
spawned uWSGI worker 1 (pid: 105734, cores: 2)
spawned uWSGI worker 2 (pid: 105735, cores: 2)
spawned uWSGI worker 3 (pid: 105736, cores: 2)
spawned uWSGI worker 4 (pid: 105737, cores: 2)
spawned uWSGI http 1 (pid: 105738)
~~~

### Base de datos

Lo primero que tenemos que hacer es crear un usuario para que administre la base de datos de `mezzanine`, como nosotros ya tenemos acceso a la base de datos de `sancho` remotamente, haremos lo siguiente desde `quijote`:

~~~
[centos@quijote ~]$ mysql -h bd.cabezas.gonzalonazareno.org -u ale -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 36
Server version: 10.3.25-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'user_mezzanine'@'%' IDENTIFIED BY 'pass_mezzanine';
~~~

Creamos la base de datos la cual servirá para mezzanine:

~~~
MariaDB [(none)]> create database bd_mezzanine;
~~~

Y modificamos los datos de acceso de mezzanine:

~~~
sudo nano mezzanine/mysite/mysite/settings.py
~~~

Y editamos la base de datos, usuario, contraseña y la ip del host `sancho`, dejándolo de la siguiente forma:

~~~
DATABASES = {
    "default": {
        # Add "postgresql_psycopg2", "mysql", "sqlite3" or "oracle".
        "ENGINE": "django.db.backends.mysql",
        # DB name or path to database file if using sqlite3.
        "NAME": "bd_mezzanine",
        # Not used with sqlite3.
        "USER": "user_mezzanine",
        # Not used with sqlite3.
        "PASSWORD": "pass_mezzanine",
        # Set to empty string for localhost. Not used with sqlite3.
        "HOST": "10.0.1.13",
        # Set to empty string for default. Not used with sqlite3.
        "PORT": "",
    }
}
~~~

También tenemos que instalar el conector de la base de datos con python:

~~~
pip install mysql-connector-python
~~~

Para crear la base de datos ejecutamos el siguiente comando:

~~~
(mezzanine) [centos@quijote mysite]$ python3 manage.py migrate
Traceback (most recent call last):
  File "manage.py", line 14, in <module>
    execute_from_command_line(sys.argv)
  File "/home/centos/entornos/mezzanine/lib64/python3.6/site-packages/django/core/management/__init__.py", line 364, in execute_from_command_line
    utility.execute()
  File "/home/centos/entornos/mezzanine/lib64/python3.6/site-packages/django/core/management/__init__.py", line 356, in execute
    self.fetch_command(subcommand).run_from_argv(self.argv)
  File "/home/centos/entornos/mezzanine/lib64/python3.6/site-packages/django/core/management/__init__.py", line 206, in fetch_command
    klass = load_command_class(app_name, subcommand)
  File "/home/centos/entornos/mezzanine/lib64/python3.6/site-packages/django/core/management/__init__.py", line 40, in load_command_class
    module = import_module('%s.management.commands.%s' % (app_name, name))
  File "/usr/lib64/python3.6/importlib/__init__.py", line 126, in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
  File "<frozen importlib._bootstrap>", line 994, in _gcd_import
  File "<frozen importlib._bootstrap>", line 971, in _find_and_load
  File "<frozen importlib._bootstrap>", line 955, in _find_and_load_unlocked
  File "<frozen importlib._bootstrap>", line 665, in _load_unlocked
  File "<frozen importlib._bootstrap_external>", line 678, in exec_module
  File "<frozen importlib._bootstrap>", line 219, in _call_with_frames_removed
  File "/home/centos/entornos/mezzanine/lib64/python3.6/site-packages/django/core/management/commands/migrate.py", line 15, in <module>
    from django.db.migrations.autodetector import MigrationAutodetector
  File "/home/centos/entornos/mezzanine/lib64/python3.6/site-packages/django/db/migrations/autodetector.py", line 13, in <module>
    from django.db.migrations.questioner import MigrationQuestioner
  File "/home/centos/entornos/mezzanine/lib64/python3.6/site-packages/django/db/migrations/questioner.py", line 12, in <module>
    from .loader import MigrationLoader
  File "/home/centos/entornos/mezzanine/lib64/python3.6/site-packages/django/db/migrations/loader.py", line 10, in <module>
    from django.db.migrations.recorder import MigrationRecorder
  File "/home/centos/entornos/mezzanine/lib64/python3.6/site-packages/django/db/migrations/recorder.py", line 12, in <module>
    class MigrationRecorder(object):
  File "/home/centos/entornos/mezzanine/lib64/python3.6/site-packages/django/db/migrations/recorder.py", line 26, in MigrationRecorder
    class Migration(models.Model):
  File "/home/centos/entornos/mezzanine/lib64/python3.6/site-packages/django/db/migrations/recorder.py", line 27, in Migration
    app = models.CharField(max_length=255)
  File "/home/centos/entornos/mezzanine/lib64/python3.6/site-packages/django/db/models/fields/__init__.py", line 1061, in __init__
    super(CharField, self).__init__(*args, **kwargs)
  File "/home/centos/entornos/mezzanine/lib64/python3.6/site-packages/django/db/models/fields/__init__.py", line 172, in __init__
    self.db_tablespace = db_tablespace or settings.DEFAULT_INDEX_TABLESPACE
  File "/home/centos/entornos/mezzanine/lib64/python3.6/site-packages/django/conf/__init__.py", line 56, in __getattr__
    self._setup(name)
  File "/home/centos/entornos/mezzanine/lib64/python3.6/site-packages/django/conf/__init__.py", line 41, in _setup
    self._wrapped = Settings(settings_module)
  File "/home/centos/entornos/mezzanine/lib64/python3.6/site-packages/django/conf/__init__.py", line 129, in __init__
    raise ImproperlyConfigured("The SECRET_KEY setting must not be empty.")
django.core.exceptions.ImproperlyConfigured: The SECRET_KEY setting must not be empty.
~~~

A nosotros nos ha dado un error de SECRET_KEY, el cuál nos dice que no tenemos una clave en el fichero _settings.py_, para poder solucionar dicho problema, vamos a generar una clave en la página https://miniwebtool.com/django-secret-key-generator/ y la pondremos en el fichero:

~~~
sudo nano mysite/mysite/settings.py
~~~

De la siguiente forma:

~~~
SECRET_KEY= ")cgg_^ud70nlh)(7-4r6k+n*9-isa6fpg64^9^+*i7z6cv3j^c"
~~~

Volvemos a ejecutar el comando para crear la base de datos:

~~~
python manage.py migrate
~~~

Y si nos da el siguiente error:

~~~
'Did you install mysqlclient or MySQL-python?' % e
django.core.exceptions.ImproperlyConfigured: Error loading MySQLdb module: No module named 'MySQLdb'.
Did you install mysqlclient or MySQL-python?
~~~

Lo podemos solucionar instalando lo siguiente:

~~~
sudo dnf install mysql-devel
pip3 install mysqlclient
~~~

Una vez resuelto, ejecutamos el comando anterior y cargamos la copia de la base de datos que hicimos en desarrollo:

~~~
python manage.py migrate
python manage.py loaddata backup_db.json 
~~~

## Despliegue en httpd

### Fichero virtualhost

Lo primero que tenemos que crear es un nuevo virtualhost, para ello (hay que tener en cuenta que hemos creado la estructura de carpetas igual que trae apache por defecto, que en centos no la tiene, y hemos tenido que modificar el fichero _/etc/httpd/conf/httpd.conf_ añadiendole la siguiente línea: _IncludeOptional sites-enabled/*.conf_):

~~~
sudo nano /etc/httpd/sites-available/mezzanine.conf
~~~

Y añadimos lo siguiente:

~~~
<VirtualHost *:80>
    ServerName python.cabezas.gonzalonazareno.org
    DocumentRoot /var/www/mezzanine/mysite

    <Proxy "unix:/run/php-fpm/www.sock|fcgi://php-fpm">
           ProxySet disablereuse=off
    </Proxy>

    <FilesMatch \.php$>
           SetHandler proxy:fcgi://php-fpm
    </FilesMatch>

    Alias /static "/var/www/mezzanine/mysite/static"

    <Directory /var/www/mezzanine/mysite/static>
           Require all granted
    </Directory>

     ProxyPass /static !
     ProxyPass / http://127.0.0.1:8080/
</VirtualHost>
~~~

Activamos el virtualhost:

~~~
sudo ln -s /etc/httpd/sites-available/mezzanine.conf /etc/httpd/sites-enabled/
~~~

### Modificacion fichero settings.py

En el `settings.py` tendremos que modificar lo siguiente, para ello:

~~~
sudo nano /var/www/mezzanine/mysite/mysite/settings.py
~~~

Modificando los siguientes parámetros para que se pueda acceder a la página y apache pueda obtener las hojas de estilo en dicha carpeta:

~~~
ALLOWED_HOSTS = ['127.0.0.1']
STATIC_ROOT = '/var/www/mezzanine/mysite/static/'
~~~

### Fichero uwsgi.ini

Para ejecutar el servidor uwsgi, crearemos el fichero:

~~~
nano uwsgi.ini
~~~

Con el siguiente contenido:

~~~
[uwsgi]
http = :8080
chdir = /var/www/mezzanine/mysite
wsgi-file = mysite/wsgi.py
processes = 4
threads = 2
~~~

Y para ejecutarlo:

~~~
uwsgi --ini uwsgi.ini
~~~

En ese momento ya debemos de tener acceso a la página pero sin mostrarnos las hojas de estilo, ya que estas nos la va a servir el servidor apache tal como le hemos indicado en el fichero virtualhost, así que para que apache pueda obtener las hojas de estilo (con el entorno virtual activo):

~~~
python3 manage.py collectstatic
~~~

Si el paso anterior nos da error, cambiamos el propietario de la carpeta `/var/www/mezzanine`:

~~~
sudo chown -R centos. /var/www/mezzanine
~~~

## Cambio en DNS en freston

Para que la página sea ofrecida por nuestro servidor DNS, tendremos que hacer la siguiente gestión en `freston`:

~~~
sudo nano /var/cache/bind/db.externa.cabezas.gonzalonazareno.org
~~~

Y añadimos lo siguiente:

~~~
python          IN      CNAME   dulcinea
~~~

## Comprobación

Comprobamos la página desde el navegador:

![7](/assets/img/posts/mezzanine/7.png)

![8](/assets/img/posts/mezzanine/8.png)