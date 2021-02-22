---
layout: post
title: Página estática con GitHub Pages y Jekyll
tags: [all, Github Pages, Jekyll]
---
# Introducción

Buenas, en esta práctica vamos a hacer la implantación y despliegue de una página web estática usando GitHub como repositorio con GitHub Pages y Jekyll. Aunque GitHub Pages funciona internamente generando Jekyll, nosotros lo haremos de forma local.

## Instalación Ruby y Bundler

Para poder usar Jekyll, primero debemos de instalar Ruby y sus dependencias. Jekyll usa el lenguaje de Ruby y el sistema de plantillas Liquid.

~~~
sudo apt install ruby-full build-essential zlib1g-dev
~~~

Posteriormente, instalamos Bundler. Bundler es un gestor de dependencias para Ruby.

~~~
sudo apt install bundler jekyll
~~~

Configuramos un directorio de instalación de gemas para nuestro usuario(sólo si queremos tenerlo más organizado):

~~~
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
~~~

## Creación de repositorio en GitHub

Nosotros vamos a crear dos repositorios en GitHub, uno será para tener los archivos necesarios para generar nuestra página y el otro para subir la página generada por jekyll.

Creamos el que tendrá la página generada:

*Nombredeusuario* .github.io

![img17](/assets/img/posts/jekyll/img17.png)

El otro le pondremos el nombre de *proyecto_jekyll*.

Clonamos en nuestro directorio de nuestro equipo:

~~~
git clone git@github.com:acabe10/acabe10.github.io.git
git clone git@github.com:acabe10/proyecto_jekyll
~~~

## Creación de sitio

Crearemos todos los archivos necesarios en *proyecto_jekyll*:

~~~
cd proyecto_jekyll
~~~

Hacemos un nuevo sitio en un nuevo direcorio llamado *docs*, que es donde vamos a tener alojada nuestra página:

~~~
jekyll new docs
cd docs
~~~

Instalamos las gem de bundler y jekyll:

~~~
gem install jekyll bundler
~~~

Para probarlo en local:

Primero cambiamos el archivo *_config.yml* con nuestros datos:

~~~
title:              Blog acabe10
email:              alejandrocabezab@gmail.com
description:        Blog sobre informática
author:             Alejandro Cabezas
baseurl:            "/"
url:                "https://acabe10.github.io"
~~~

Y en el *GemFile* podemos añadir nuevos plugins, gemas.. etc. Para actualizar gemas y ficheros:

~~~
bundle update
~~~

Para generar la página:

~~~
bundle exec jekyll serve
~~~

Y accedemos a *localhost:4000* desde el navegador:

![img18](/assets/img/posts/jekyll/img18.png)

Subo al repositorio remoto:

~~~
git add *
git commit -m "añado new jekyll"
git push
~~~

## Cambiar tema

Hay varias formas de cambiar el tema de nuestro sitio, clonando un repositorio remoto, descargando la plantilla, instalando una gema.. nosotros 
lo hemos hecho mediante la descarga de la plantilla, pero explico como hacerlo por un repositorio remoto:

Nos vamos al archivo *GemFile* y comentamos la siguiente línea:

~~~
gem "minima", "~> 2.5"
~~~

Y descomentamos esta otra:

~~~
gem "github-pages", group: :jekyll_plugins
~~~

Y en el archivo *_config.yml* comentamos la línea de nuestro tema por defecto y añadimos la siguiente:

~~~
remote_theme: "StartBootstrap/startbootstrap-clean-blog-jekyll"
~~~

Hay bastantes páginas para poder obtener temas directamente desde *github*:

[Temas jekyll](https://github.com/topics/jekyll-theme)

Ahora ejecutamos:

~~~
bundle update
~~~

Y ya tendríamos nuestra plantilla incorporada. Ahora sólo nos falta terminar de añadir las páginas que faltan y los enlaces hacia otras páginas,
 en la página de donde hemos sacado nuestra plantilla nos explica lo que tenemos que hacer para terminar de completarla:

[Clean Blog jekyll](https://github.com/StartBootstrap/startbootstrap-clean-blog-jekyll)


## Cambiar background de cualquier página

Para cambiar el fondo de alguna página, tenemos que irnos a su *html* y editar el *background*:

~~~
nano index.html

background: '/img/bg-index.jpg'
~~~

## Crear un post

Para crear un post basta con hacer un documento en .md y colocarlo en *_posts* , jekyll se encargará de transformarlo a *html* , lo único que tenemos que tener en cuenta es que el nombre del archivo tiene que tener la fecha en este formato:

~~~
2020-10-08-Nombre-post-con-guiones-como-espacios.md
~~~

Al crear el post, le pondremos la siguiente cabecera para que coja la plantilla por defecto y el fondo:

~~~
---
layout: post
background: '/img/bg-post.jpg'
---
~~~

## Cambiar barra de navegación

Para ello, tenemos que entender como funcionan las páginas, en *_layouts* tenemos nuestros 4 diseños los cuáles que van incluyendo por módulos la cabecera, cuerpo, etc..

Y esto coge su contenido de *includes*, en el cuál se encuentran la cabezera, el pie de página, la páginación...etc. Entonces el archivo que tenemos que modificar sería el *includes/navbar.html*.

## GitIgnore

Para que determinados archivos no se tengan que sincronizar con GitHub vamos a crear fichero *.gitignore* y añadimos lo que queramos omitir:

~~~
_site/
.jekyll-cache/
~~~

Ahora cuando ejecutemos:

~~~
bundle exec jekyll serve
~~~

Se generará nuestra página en local pero cuando hagamos un git push no se nos subirán los archivos generados.

## Proceso despliegue GitHub Pages

GitHub Pages toma los archivos HTML, CSS y JavaScript directamente desde nuestro repositorio de GitHub y opcionalmente ejecuta los archivos a través de un proceso de compilación y publica un sitio web. Nosotros en cambio el sitio lo vamos a generar en local con Jekyll y a GitHub le subiremos los archivos HTML ya generados. Lo veremos en el siguiente apartado.

## Automatización página

Para automatizar la subida de archivos a git a nuestro otro repositorio vamos a crear el siguiente script y lo colocaremos en */usr/local/bin* con permisos de ejecución:

~~~
#!/bin/bash

# Comprobamos si el directorio en el que estamos es de un repositorio git
if [ ! -d '.git' ]; then
        echo 'Esta carpeta no contiene un repositorio Git'
        exit -1
fi

# Indicamos a Git los archivos a subir
git add *

# Esto nos pedira el mensaje del commit
echo "Introduce el mensaje del commit:"
read txt
git commit -am "$txt"

# Subimos los archivos a proyecto_jekyll
git push origin master

# Generamos jekyll
cd docs/
bundle exec jekyll build

# Copiamos _site jekyll a otro repositorio
cp -R _site/* /home/ale/github/acabe10.github.io

# Subimos a github
cd /home/ale/github/acabe10.github.io
git add *
git commit -am "$txt"
git push origin master
~~~

Y cuando hagamos algún cambio en nuestra página, bastará con ejecutar:

~~~
uptogit docs/
~~~

El script lo que hará será generar una página con jekyll y copiar los archivos al repositorio de GitHub que mostrará dicha página.