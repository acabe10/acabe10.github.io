---
layout: post
title: Introducción a la integración continua
tags: [all, Integración continua, IC/DC, Despliegue]
---
# Introducción

Buenas, en este post vamos a usar para un despliegue continuo el sitio web de Netlify. Tendremos el escenario que creamos en la [primera práctica](https://acabe10.github.io/2021-02-16-P%C3%A1gina-est%C3%A1tica-con-Github-Pages-y-Jekyll/) y con ello Netlify desplegará a través de Jekyll una página estática en su dominio `netlify.app`:

# Creación de sitio y despliegue

Nos hemos creado una cuenta en Netlify y hemos creado un nuevo `team`. Ahora creamos un `New site from Git`:

![1](/assets/img/posts/ic/1.png)

Seleccionamos que lo vamos a usar con una cuenta de GitHub:

![2](/assets/img/posts/ic/2.png)

Nos pedirá que instalemos Netlify en nuestro repositorio de GitHub para la integración:

![3](/assets/img/posts/ic/3.png)

Una vez instalado podremos seleccionar nuestro repositorio elegido:

![4](/assets/img/posts/ic/4.png)

Le decimos cuál es el comando que se va a ejecutar para la integración continua, en este caso, nosotros haremos un `jekyll build` para que cree el sitio, y el directorio que queremos que despliegue en producción será la carpeta site, que es la que genera el `jekyll build`:

![5](/assets/img/posts/ic/5.png)

Una vez desplegada, vemos que la página no tiene el nombre que nos gustaría y que además ha habido un fallo, para solucionar lo del nombre pulsamos en `Site settings`:

![6](/assets/img/posts/ic/6.png)

Y pulsamos en `Change site name`:

![7](/assets/img/posts/ic/7.png)

Quedando nuestro nombre de la siguiente forma:

![8](/assets/img/posts/ic/8.png)

Para solucionar el error de deploy anterior vamos a irnos al apartado `deploys`, y seleccionamos `deploy settings`:

![10](/assets/img/posts/ic/10.png)

Editamos las opciones ya que si nos fijamos no tenemos nada en `Base directory`, y nosotros en nuestro repositorio tenemos la siguiente estructura:

~~~
└── proyecto_jekyll
    ├── docs
    └── README.md
~~~

![11](/assets/img/posts/ic/11.png)

Ponemos el directorio `docs` como base directory:

![12](/assets/img/posts/ic/12.png)

Hecho lo anterior, volvemos a desplegar:

![13](/assets/img/posts/ic/13.png)

Empezará el despliegue:

![14](/assets/img/posts/ic/14.png)

Y continuará de la siguiente forma:

~~~
6:44:48 PM: Build ready to start
6:44:49 PM: build-image version: d84c79427e8f83c1ba17bcdd7b3fe38059376b68
6:44:49 PM: build-image tag: v3.6.1
6:44:49 PM: buildbot version: a0f5a242397f47e25a65cd8b1de9758023166a1b
6:44:50 PM: Fetching cached dependencies
6:44:50 PM: Failed to fetch cache, continuing with build
6:44:50 PM: Starting to prepare the repo for build
6:44:50 PM: No cached dependencies found. Cloning fresh repo
6:44:50 PM: git clone https://github.com/acabe10/proyecto_jekyll
6:44:52 PM: Preparing Git Reference refs/heads/master
6:44:53 PM: Different publish path detected, going to use the one specified in the Netlify configuration file: 'docs/_site' versus '_site/' in the Netlify UI
6:44:53 PM: Starting build script
6:44:53 PM: Installing dependencies
6:44:53 PM: Python version set to 2.7
6:44:54 PM: v12.18.0 is already installed.
6:44:55 PM: Now using node v12.18.0 (npm v6.14.4)
6:44:55 PM: Started restoring cached build plugins
6:44:55 PM: Finished restoring cached build plugins
6:44:55 PM: Attempting ruby version 2.7.1, read from environment
6:44:56 PM: Using ruby version 2.7.1
6:44:56 PM: Using bundler version 2.2.8 from Gemfile.lock
6:44:57 PM: Successfully installed bundler-2.2.8
6:44:57 PM: 1 gem installed
6:44:57 PM: Using PHP version 5.6
6:44:57 PM: Started restoring cached ruby gems
6:44:57 PM: Finished restoring cached ruby gems
6:44:57 PM: Installing gem bundle
6:44:57 PM: [DEPRECATED] The `--path` flag is deprecated because it relies on being remembered across bundler invocations, which bundler will no longer do in future versions. Instead please use `bundle config set --local path '/opt/build/cache/bundle'`, and stop using this flag
6:44:57 PM: [DEPRECATED] The --binstubs option will be removed in favor of `bundle binstubs --all`
6:44:59 PM: Fetching gem metadata from https://rubygems.org/.........
6:44:59 PM: Using bundler 2.2.8
6:44:59 PM: Fetching eventmachine 1.2.7
6:44:59 PM: Fetching colorator 1.1.0
6:44:59 PM: Fetching concurrent-ruby 1.1.8
6:44:59 PM: Fetching forwardable-extended 2.6.0
6:44:59 PM: Fetching http_parser.rb 0.6.0
6:44:59 PM: Fetching ffi 1.14.2
6:44:59 PM: Fetching public_suffix 4.0.6
6:44:59 PM: Fetching rb-fsevent 0.10.4
6:44:59 PM: Installing forwardable-extended 2.6.0
6:44:59 PM: Installing colorator 1.1.0
6:44:59 PM: Installing rb-fsevent 0.10.4
6:44:59 PM: Installing public_suffix 4.0.6
6:44:59 PM: Installing http_parser.rb 0.6.0 with native extensions
6:44:59 PM: Fetching rexml 3.2.4
6:44:59 PM: Fetching liquid 4.0.3
6:44:59 PM: Installing concurrent-ruby 1.1.8
6:44:59 PM: Installing eventmachine 1.2.7 with native extensions
6:44:59 PM: Fetching mercenary 0.4.0
6:44:59 PM: Installing rexml 3.2.4
6:44:59 PM: Installing ffi 1.14.2 with native extensions
6:44:59 PM: Installing liquid 4.0.3
6:44:59 PM: Installing mercenary 0.4.0
6:45:00 PM: Fetching rouge 3.26.0
6:45:00 PM: Fetching safe_yaml 1.0.5
6:45:00 PM: Installing rouge 3.26.0
6:45:00 PM: Installing safe_yaml 1.0.5
6:45:13 PM: Fetching unicode-display_width 1.7.0
6:45:13 PM: Fetching pathutil 0.16.2
6:45:13 PM: Fetching jekyll-paginate 1.1.0
6:45:13 PM: Installing jekyll-paginate 1.1.0
6:45:13 PM: Installing unicode-display_width 1.7.0
6:45:13 PM: Installing pathutil 0.16.2
6:45:20 PM: Fetching addressable 2.7.0
6:45:20 PM: Installing addressable 2.7.0
6:45:20 PM: Fetching i18n 1.8.8
6:45:20 PM: Fetching kramdown 2.3.0
6:45:20 PM: Fetching terminal-table 2.0.0
6:45:20 PM: Fetching sassc 2.4.0
6:45:20 PM: Fetching rb-inotify 0.10.1
6:45:20 PM: Fetching em-websocket 0.5.2
6:45:20 PM: Installing rb-inotify 0.10.1
6:45:21 PM: Installing terminal-table 2.0.0
6:45:21 PM: Installing i18n 1.8.8
6:45:21 PM: Fetching listen 3.4.1
6:45:21 PM: Installing em-websocket 0.5.2
6:45:21 PM: Installing kramdown 2.3.0
6:45:21 PM: Installing sassc 2.4.0 with native extensions
6:45:21 PM: Installing listen 3.4.1
6:45:21 PM: Fetching jekyll-watch 2.2.1
6:45:21 PM: Installing jekyll-watch 2.2.1
6:48:17 PM: Fetching kramdown-parser-gfm 1.1.0
6:48:17 PM: Installing kramdown-parser-gfm 1.1.0
6:48:17 PM: Fetching jekyll-sass-converter 2.1.0
6:48:17 PM: Installing jekyll-sass-converter 2.1.0
6:48:18 PM: Fetching jekyll 4.2.0
6:48:18 PM: Installing jekyll 4.2.0
6:48:18 PM: Fetching jekyll-feed 0.15.1
6:48:18 PM: Fetching jekyll-sitemap 1.4.0
6:48:18 PM: Installing jekyll-feed 0.15.1
6:48:18 PM: Installing jekyll-sitemap 1.4.0
6:48:18 PM: Bundle complete! 5 Gemfile dependencies, 31 gems now installed.
6:48:18 PM: Bundled gems are installed into `/opt/build/cache/bundle`
6:48:18 PM: Gem bundle installed
6:48:18 PM: Started restoring cached node modules
6:48:18 PM: Finished restoring cached node modules
6:48:18 PM: Installing NPM modules using NPM version 6.14.4
6:48:27 PM: npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.9 (node_modules/fsevents):
6:48:27 PM: npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.9: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"x64"})
6:48:27 PM: added 455 packages from 363 contributors and audited 523 packages in 8.496s
6:48:28 PM: 1 package is looking for funding
6:48:28 PM:   run `npm fund` for details
6:48:28 PM: found 11 vulnerabilities (8 low, 3 high)
6:48:28 PM:   run `npm audit fix` to fix them, or `npm audit` for details
6:48:28 PM: NPM modules installed
6:48:28 PM: Started restoring cached go cache
6:48:28 PM: Finished restoring cached go cache
6:48:28 PM: go version go1.14.4 linux/amd64
6:48:28 PM: go version go1.14.4 linux/amd64
6:48:28 PM: Installing missing commands
6:48:28 PM: Verify run directory
6:48:29 PM: ​
6:48:29 PM: ────────────────────────────────────────────────────────────────
6:48:29 PM:   Netlify Build                                                 
6:48:29 PM: ────────────────────────────────────────────────────────────────
6:48:29 PM: ​
6:48:29 PM: ❯ Version
6:48:29 PM:   @netlify/build 8.3.4
6:48:29 PM: ​
6:48:29 PM: ❯ Flags
6:48:29 PM:   deployId: 601ae110b14f5b008df35ce5
6:48:29 PM:   mode: buildbot
6:48:29 PM: ​
6:48:29 PM: ❯ Current directory
6:48:29 PM:   /opt/build/repo/docs
6:48:29 PM: ​
6:48:29 PM: ❯ Config file
6:48:29 PM:   No config file was defined: using default values.
6:48:29 PM: ​
6:48:29 PM: ❯ Context
6:48:29 PM:   production
6:48:29 PM: ​
6:48:29 PM: ────────────────────────────────────────────────────────────────
6:48:29 PM:   1. Build command from Netlify app                             
6:48:29 PM: ────────────────────────────────────────────────────────────────
6:48:29 PM: ​
6:48:29 PM: $ bundle exec jekyll build
6:48:30 PM: Configuration file: /opt/build/repo/docs/_config.yml
6:48:30 PM:             Source: /opt/build/repo/docs
6:48:30 PM:        Destination: /opt/build/repo/docs/_site
6:48:30 PM:  Incremental build: disabled. Enable with --incremental
6:48:30 PM:       Generating...
6:48:30 PM:        Jekyll Feed: Generating feed for posts
6:48:31 PM:                     done in 0.509 seconds.
6:48:31 PM:  Auto-regeneration: disabled. Use --watch to enable.
6:48:31 PM: ​
6:48:31 PM: (build.command completed in 1.2s)
6:48:31 PM: ​
6:48:31 PM: ────────────────────────────────────────────────────────────────
6:48:31 PM:   2. Deploy site                                                
6:48:31 PM: ────────────────────────────────────────────────────────────────
6:48:31 PM: ​
6:48:31 PM: Starting to deploy site from 'docs/_site'
6:48:31 PM: Creating deploy tree asynchronously
6:48:31 PM: Creating deploy upload records
6:48:33 PM: 1 new files to upload
6:48:33 PM: 0 new functions to upload
6:48:33 PM: Site deploy was successfully initiated
6:48:33 PM: ​
6:48:33 PM: (Deploy site completed in 2.5s)
6:48:33 PM: ​
6:48:33 PM: ────────────────────────────────────────────────────────────────
6:48:33 PM:   Netlify Build Complete                                        
6:48:33 PM: ────────────────────────────────────────────────────────────────
6:48:33 PM: ​
6:48:33 PM: (Netlify Build completed in 3.8s)
6:48:33 PM: Caching artifacts
6:48:33 PM: Started saving ruby gems
6:48:33 PM: Finished saving ruby gems
6:48:33 PM: Started saving node modules
6:48:33 PM: Finished saving node modules
6:48:33 PM: Started saving build plugins
6:48:33 PM: Finished saving build plugins
6:48:33 PM: Started saving pip cache
6:48:33 PM: Finished saving pip cache
6:48:33 PM: Started saving emacs cask dependencies
6:48:33 PM: Finished saving emacs cask dependencies
6:48:33 PM: Started saving maven dependencies
6:48:33 PM: Finished saving maven dependencies
6:48:33 PM: Started saving boot dependencies
6:48:33 PM: Finished saving boot dependencies
6:48:33 PM: Started saving rust rustup cache
6:48:33 PM: Finished saving rust rustup cache
6:48:33 PM: Started saving go dependencies
6:48:33 PM: Finished saving go dependencies
6:48:34 PM: Starting post processing
6:48:34 PM: Post processing - HTML
6:48:34 PM: Post processing - header rules
6:48:34 PM: Post processing - redirect rules
6:48:34 PM: Post processing done
6:48:34 PM: Site is live ✨
6:48:36 PM: Build script success
6:48:58 PM: Finished processing build request in 4m8.263018776s
~~~

Una vez acabado podemos irnos a la url https://acabe10.netlify.app para ver que funciona correctamente:

![15](/assets/img/posts/ic/15.png)

# Modificación repositorio

Para comprobar que cuando hacemos un `commit` y un `push` automáticamente hace la integración en la página de netlify vamos a modificar el título de la página:

~~~
ale@arya  ~/github/proyecto_jekyll/docs   master  nano _config.yml
~~~

Añadimos lo siguiente:

~~~
title:              Probando IC  
email:              alejandrocabezab@gmail.com
description:        Blog sobre informática
author:             Alejandro Cabezas
baseurl:            "/"
url:                "https://acabe10.github.io"
~~~

Y hacemos la subida a github:

~~~
git commit -am "probando ic"
git push
~~~

Si nos vamos a `Netlify` podemos comprobarlo:

![16](/assets/img/posts/ic/16.png)

Este es el `build` completo:

~~~
6:52:36 PM: Build ready to start
6:52:37 PM: build-image version: d84c79427e8f83c1ba17bcdd7b3fe38059376b68
6:52:37 PM: build-image tag: v3.6.1
6:52:37 PM: buildbot version: a0f5a242397f47e25a65cd8b1de9758023166a1b
6:52:37 PM: Fetching cached dependencies
6:52:38 PM: Starting to download cache of 163.9MB
6:52:38 PM: Finished downloading cache in 981.276205ms
6:52:38 PM: Starting to extract cache
6:52:43 PM: Finished extracting cache in 4.408056636s
6:52:43 PM: Finished fetching cache in 5.426911088s
6:52:43 PM: Starting to prepare the repo for build
6:52:43 PM: Preparing Git Reference refs/heads/master
6:52:45 PM: Different publish path detected, going to use the one specified in the Netlify configuration file: 'docs/_site' versus '_site/' in the Netlify UI
6:52:45 PM: Starting build script
6:52:45 PM: Installing dependencies
6:52:45 PM: Python version set to 2.7
6:52:46 PM: Started restoring cached node version
6:52:48 PM: Finished restoring cached node version
6:52:49 PM: v12.18.0 is already installed.
6:52:49 PM: Now using node v12.18.0 (npm v6.14.4)
6:52:49 PM: Started restoring cached build plugins
6:52:49 PM: Finished restoring cached build plugins
6:52:50 PM: Attempting ruby version 2.7.1, read from environment
6:52:51 PM: Using ruby version 2.7.1
6:52:51 PM: Using bundler version 2.2.8 from Gemfile.lock
6:52:52 PM: Successfully installed bundler-2.2.8
6:52:52 PM: 1 gem installed
6:52:52 PM: Using PHP version 5.6
6:52:52 PM: Started restoring cached ruby gems
6:52:52 PM: Finished restoring cached ruby gems
6:52:52 PM: Started restoring cached node modules
6:52:52 PM: Finished restoring cached node modules
6:52:52 PM: Started restoring cached go cache
6:52:52 PM: Finished restoring cached go cache
6:52:52 PM: go version go1.14.4 linux/amd64
6:52:52 PM: go version go1.14.4 linux/amd64
6:52:52 PM: Installing missing commands
6:52:52 PM: Verify run directory
6:52:54 PM: ​
6:52:54 PM: ────────────────────────────────────────────────────────────────
6:52:54 PM:   Netlify Build                                                 
6:52:54 PM: ────────────────────────────────────────────────────────────────
6:52:54 PM: ​
6:52:54 PM: ❯ Version
6:52:54 PM:   @netlify/build 8.3.4
6:52:54 PM: ​
6:52:54 PM: ❯ Flags
6:52:54 PM:   deployId: 601ae2e364508b000719e7c1
6:52:54 PM:   mode: buildbot
6:52:54 PM: ​
6:52:54 PM: ❯ Current directory
6:52:54 PM:   /opt/build/repo/docs
6:52:54 PM: ​
6:52:54 PM: ❯ Config file
6:52:54 PM:   No config file was defined: using default values.
6:52:54 PM: ​
6:52:54 PM: ❯ Context
6:52:54 PM:   production
6:52:54 PM: ​
6:52:54 PM: ────────────────────────────────────────────────────────────────
6:52:54 PM:   1. Build command from Netlify app                             
6:52:54 PM: ────────────────────────────────────────────────────────────────
6:52:54 PM: ​
6:52:54 PM: $ bundle exec jekyll build
6:52:55 PM: Configuration file: /opt/build/repo/docs/_config.yml
6:52:55 PM:             Source: /opt/build/repo/docs
6:52:55 PM:        Destination: /opt/build/repo/docs/_site
6:52:55 PM:  Incremental build: disabled. Enable with --incremental
6:52:55 PM:       Generating...
6:52:55 PM:        Jekyll Feed: Generating feed for posts
6:52:55 PM:                     done in 0.493 seconds.
6:52:55 PM:  Auto-regeneration: disabled. Use --watch to enable.
6:52:55 PM: ​
6:52:55 PM: (build.command completed in 1.3s)
6:52:55 PM: ​
6:52:55 PM: ────────────────────────────────────────────────────────────────
6:52:55 PM:   2. Deploy site                                                
6:52:55 PM: ────────────────────────────────────────────────────────────────
6:52:55 PM: ​
6:52:55 PM: Starting to deploy site from 'docs/_site'
6:52:55 PM: Creating deploy tree asynchronously
6:52:55 PM: Creating deploy upload records
6:52:58 PM: 8 new files to upload
6:52:58 PM: 0 new functions to upload
6:52:58 PM: Site deploy was successfully initiated
6:52:58 PM: ​
6:52:58 PM: (Deploy site completed in 2.6s)
6:52:58 PM: ​
6:52:58 PM: ────────────────────────────────────────────────────────────────
6:52:58 PM:   Netlify Build Complete                                        
6:52:58 PM: ────────────────────────────────────────────────────────────────
6:52:58 PM: ​
6:52:58 PM: (Netlify Build completed in 4s)
6:52:58 PM: Caching artifacts
6:52:58 PM: Started saving ruby gems
6:52:58 PM: Finished saving ruby gems
6:52:58 PM: Started saving node modules
6:52:58 PM: Finished saving node modules
6:52:58 PM: Started saving build plugins
6:52:58 PM: Finished saving build plugins
6:52:58 PM: Started saving pip cache
6:52:58 PM: Finished saving pip cache
6:52:58 PM: Started saving emacs cask dependencies
6:52:58 PM: Finished saving emacs cask dependencies
6:52:58 PM: Started saving maven dependencies
6:52:58 PM: Finished saving maven dependencies
6:52:58 PM: Started saving boot dependencies
6:52:58 PM: Finished saving boot dependencies
6:52:58 PM: Started saving rust rustup cache
6:52:58 PM: Finished saving rust rustup cache
6:52:58 PM: Started saving go dependencies
6:52:58 PM: Finished saving go dependencies
6:52:58 PM: Build script success
6:52:58 PM: Starting post processing
6:52:58 PM: Post processing - HTML
6:52:59 PM: Post processing - header rules
6:52:59 PM: Post processing - redirect rules
6:52:59 PM: Post processing done
6:52:59 PM: Site is live ✨
~~~

Y una vez realizado el `build` correctamente, podemos comprobarlo en la página:

![17](/assets/img/posts/ic/17.png)