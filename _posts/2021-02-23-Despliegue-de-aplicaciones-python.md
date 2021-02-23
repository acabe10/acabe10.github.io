---
layout: post
title: Despliegue de aplicaciones python
tags: [all, Python, Despliegue, Aplicacion, Django]
---
# Introducción

Buenas, en este post vamos a desarrollar la aplicación del [tutorial de django 3.1.](https://docs.djangoproject.com/en/3.1/intro/tutorial01/). Primero lo haremos en un entorno de desarrollo y posteriormente en un entorno de producción.

## Entorno de desarrollo

Vamos a configurar nuestro equipo para trabajar con la aplicación:

* Realizamos un fork del repositorio: https://github.com/acabe10/django_tutorial

~~~
git clone git@github.com:acabe10/django_tutorial.git
~~~

* Creamos entorno e instalamos dependencias

~~~
python3 -m venv django 
source django/bin/activate
~~~

	- Instalamos las dependencias necesarias:

	~~~
	pip install -r requirements.txt
	~~~

* Creamos base de datos

	- La base de datos se llamará:

	~~~
	db.sqlite3
	~~~
	
	- La creamos:

	~~~
	python3 manage.py migrate
	~~~

* Creamos usuario

~~~
python3 manage.py createsuperuser
~~~

* Lanzamos el servidor en desarrollo:

~~~
python3 manage.py runserver
~~~

* Comprobación

![1](/assets/img/posts/django_tutorial/1.png)

![2](/assets/img/posts/django_tutorial/2.png)

## Entorno de producción

Suponiendo que ya tenemos una máquina creada en el cloud.

* Instalaciones necesarias

	- Instalamos <code>apache2</code> ,<code>git</code>, <code>mariadb</code> y lo necesario para crear entornos:

	~~~
	sudo apt install apache2 git python3-venv mariadb-server mariadb-client
	~~~

	- Instalamos el módulo necesario:

	~~~
	sudo apt install libapache2-mod-wsgi-py3
	~~~

	- Comprobamos que el módulo está activo:

	~~~
	sudo apache2ctl -M
	~~~

* Clonamos el repositorio

	Clonamos el repositorio en el <code>DocumentRoot</code> de nuestro virtualhost:

	~~~
	cd /var/www/html/
	sudo git clone https://github.com/acabe10/django_tutorial
	~~~

* Entorno virtual

	Creamos un nuevo entorno:

	~~~
	python3 -m venv /home/debian/django
	~~~

	- Lo activamos:

	~~~
	source /home/debian/django/bin/activate
	~~~

* Instalamos dependencias y el módulo que permite a <code>python</code> trabajar con <code>mysql</code>:

~~~
cd /var/www/html/django_tutorial
pip install -r requirements.txt
pip	install mysql-connector-python
~~~

* MariaDB

	- Accedemos y creamos usuario nuevo y bd:

	~~~
	sudo mysql -u root

	> GRANT ALL PRIVILEGES ON *.* TO 'ale'@'%' IDENTIFIED BY 'ale' WITH GRANT OPTION;flush privileges;

	> create database python3;
	~~~

* Configuración aplicación base de datos

	- Accedemos al archivo de configuración de la app:

	~~~
	sudo nano /var/www/html/django_tutorial/django_tutorial/settings.py
	~~~

	- Cambiamos la base de datos:

	~~~
	DATABASES = {
	    'default': {
	        'ENGINE': 'mysql.connector.django',
	        'NAME': 'python3',
	        'USER': 'ale',
	        'PASSWORD': 'ale',
	        'HOST': 'localhost',
	        'PORT': '',
	    }
	}
	~~~

* Migración base de datos

~~~
python3 manage.py migrate
~~~

	- Podemos confirmar viendo las tablas en nuestro usuario de _mariadb_:

	~~~
	MariaDB [python3]> show tables;
	+----------------------------+
	| Tables_in_python3          |
	+----------------------------+
	| auth_group                 |
	| auth_group_permissions     |
	| auth_permission            |
	| auth_user                  |
	| auth_user_groups           |
	| auth_user_user_permissions |
	| django_admin_log           |
	| django_content_type        |
	| django_migrations          |
	| django_session             |
	| polls_choice               |
	| polls_question             |
	+----------------------------+
	~~~

* Creación usuario

	- Creamos usuario administrador de django:

	~~~
	python3 manage.py createsuperuser

	Username (leave blank to use 'debian'): ale
	Email address: ale@ale.com
	Password: 
	Password (again): 
	The password is too similar to the username.
	This password is too short. It must contain at least 8 characters.
	This password is too common.
	Bypass password validation and create user anyway? [y/N]: y
	Superuser created successfully.
	~~~

* Configuración virtualhost

	- Abrimos el fichero de configuración de nuestro virtualhost:

	~~~
	sudo nano /etc/apache2/sites-available/000-default.conf
	~~~

	- Y lo dejamos de la siguiente forma:

	~~~
	<VirtualHost *:80>
	        ServerName www.ale-temp.com

	        ServerAdmin webmaster@localhost
	        DocumentRoot /var/www/html/django_tutorial

	        WSGIDaemonProcess django_tutorial user=www-data group=www-data processes=1 threads=5 python-path=/var/www/html/django_tutorial:/home/debian/django/lib/python3.7/site-packages
	        WSGIScriptAlias / /var/www/html/django_tutorial/django_tutorial/wsgi.py

	        <Directory /var/www/html/django_tutorial>
	                WSGIProcessGroup django_tutorial
	                WSGIApplicationGroup %{GLOBAL}
	                Require all granted
	        </Directory>

	        ErrorLog ${APACHE_LOG_DIR}/error.log
	        CustomLog ${APACHE_LOG_DIR}/access.log combined

	</VirtualHost>
	~~~

* Reiniciamos el servicio:

~~~
sudo systemctl restart apache2
~~~

* Comprobaciones

	- Si comprobamos nos sale el siguiente error:

![3](/assets/img/posts/django_tutorial/3.png)

	- Para resolverlo, tendremos que editar el fichero de configuración:

	~~~
	sudo nano /var/www/html/django_tutorial/django_tutorial/settings.py
	~~~

	- Y modificar el _ALLOWED_HOSTS_ dejándolo para que funcione tanto en desarrolo como en producción:

	~~~
	ALLOWED_HOSTS = ['www.ale-temp.com','www.ale-arya.com']
	~~~

	- Compruebo que nos carga la página, pero sin la hoja de estilo:

![4](/assets/img/posts/django_tutorial/4.png)

![5](/assets/img/posts/django_tutorial/5.png)

* Mostrar hojas de estilo

	- Para poder mostrar las hojas de estilo en django, creamos un STATIC_ROOT para decirle dónde estarán nuestros ficheros, para ello:

	~~~
	sudo nano /var/www/html/django_tutorial/django_tutorial/settings.py
	~~~

	- Y añadimos:

	~~~
	STATIC_ROOT= '/var/www/html/django_tutorial/static/'
	~~~

	- También editamos el fichero de configuración de nuestro virtualhost para que obtenga los fichero de dicho lugar:

	~~~
	sudo nano /etc/apache2/sites-available/000-default.conf
	~~~

	- Y añadimos lo siguiente:

	~~~
	Alias /static /var/www/html/django_tutorial/static

	<Directory /var/www/html/django_tutorial/static>
	                Require all granted
	</Directory>
	~~~

	- Ahora tendremos que indicarle que mueva los archivos que están en el directorio de nuestro sitios para el administrador a la ruta mencionada en el STATIC_ROOT (este paso lo tenemos que hacer con nuestro entorno virtual activo):

	~~~
	python3 manage.py collectstatic
	~~~

	- Y comprobamos que hemos obtenido los archivos:

	~~~
	ls /var/www/html/django_tutorial/static 
	admin  polls
	~~~

	- Reiniciamos el servicio:

	~~~
	sudo systemctl restart apache2
	~~~

	- Comprobamos:

![6](/assets/img/posts/django_tutorial/6.png)

* Modo debug

	- Vamos al fichero:

	~~~
	sudo nano /var/www/html/django_tutorial/django_tutorial/settings.py
	~~~

	- Y cambiamos:

	~~~
	DEBUG = False
	~~~

## Modificación de aplicación

* Añadimos nuestro nombre

	- Para que nuestro nombre aparezca arriba de la página de las encuestas modificamos:

	~~~
	sudo nano /var/www/html/django_tutorial/polls/templates/polls/index.html
	~~~

	- Y lo dejamos de la siguiente forma:

		~~~
		...
		<h1>Alejandro Cabezas</h1>
		{% if latest_question_list %}
		...
		~~~

	- Comprobamos en desarrollo:

![7](/assets/img/posts/django_tutorial/7.png)

	- Y para que los cambios sean efectivos en producción:

	~~~
	git commit -am "añado nombre"
	git push
	~~~

	- Y en producción ejecutamos:

	~~~
	sudo git pull
	~~~

	- Comprobamos en producción:

![8](/assets/img/posts/django_tutorial/8.png)

* Modificar imagen de fondo

	- En desarrollo, en la ruta:

	~~~
	/var/www/html/django_tutorial/polls/static/polls/images
	~~~

	- Añadimos una nueva imagen y comprobamos:

![9](/assets/img/posts/django_tutorial/9.png)

	- Subimos a github:

	~~~
	git commit -am "añado background"
	git push
	~~~

	- Y en producción ejecuto:

	~~~
	sudo git pull
	~~~

	- Y comprobamos:

![10](/assets/img/posts/django_tutorial/10.png)

* Crear nueva tabla

	- Nos vamos a:

	~~~
	sudo nano /var/www/html/django_tutorial/polls/models.py
	~~~

	- Y añadimos:

	~~~
	class Categoria(models.Model):	
	  	Abr = models.CharField(max_length=4)
	  	Nombre = models.CharField(max_length=50)

	  	def __str__(self):
	  		return self.Abr+" - "+self.Nombre 
	~~~

	- Ejecutamos para crear la nueva migración:

	~~~
	python3 manage.py makemigrations
	Migrations for 'polls':
	  polls/migrations/0002_categoria.py
	    - Create model Categoria
	~~~

	- Y realizamos la migración:

	~~~
	python3 manage.py migrate
	Operations to perform:
	  Apply all migrations: admin, auth, contenttypes, polls, sessions
	Running migrations:
	  Applying polls.0002_categoria... OK
	~~~

	- Ahora añadimos el nuevo modelo al sitio de administración de django, añadiendo en la segunda línea  el nombre del modelo

	~~~
	from .models import Choice, Question, Categoria
	~~~

	- Y al final de la línea:

	~~~
	admin.site.register(Categoria)
	~~~

	- Comprobamos en desarrollo:

![11](/assets/img/posts/django_tutorial/11.png)

	- Subimos a git:

	~~~
	git add polls/migrations/0002_categoria.py
	git commit -am "nuevo modelo"
	git push
	~~~

	- Y en producción:

	~~~
	sudo git pull
	~~~

	- Y comprobamos:

![12](/assets/img/posts/django_tutorial/12.png)