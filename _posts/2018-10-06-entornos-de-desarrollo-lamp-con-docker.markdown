---
layout: post
title: Entornos de desarrollo lamp con docker
date: 06/10/2018
description: En nuestro primer post vamos a configurar usando docker un entorno de desarrollo LAMP bajo Linux usando la distribución Debían. # Add post description (optional)
img: entornos-docker-lamp/docker.png # Add image post (optional)
tags: [Docker, Lamp, Linux] # add tag
---

En nuestro primer post vamos a configurar usando docker un entorno de desarrollo LAMP bajo Linux usando la distribución Debían.

Este entorno está pensado exclusivamente para el desarrollo local, no usarlo en un servidor de producción.

Las principales ventajas que tiene usar esta configuración son:


– Tener unos contenedores reaprovechables para nuestros proyectos, no tener que estar creando nuevos contenedores para cada proyecto (siempre que usen las mismas versiones de todo el stack lamp).

– Volúmenes externos para tener acceso al código de forma rápida y fácil. Podemos destruir los contenedores por cualquier motivo y cuando los levantemos de nuevo nuestras bases de datos, carpetas de proyectos, virtualhost, … seguirán intactos.

Enlace al repositorio en [github](https://github.com/n0ni0/docker-lamp){:target="_blank"}

Si deseamos instalar este entorno en un sistema operativo windows ó mac siguiento este tutorial, tendríamos que irnos a la [documentación oficial de docker](https://docs.docker.com/){:target="_blank"} , y una vez lo tengamos instalado, clonarnos el repositorio y seguir los pasos.
Recomiendo añadir una herramienta de sincronización de archivos tipo dockersync para mejorar el rendimiento en mac y windows, pues se nota bastante debido a la conversión de archivos que tiene que realizar.

Si trabajamos desde linux no tendremos este problema

### Requisitos previos ###
Si tenemos corriendo alguna instancia de apache, php ó mysql es recomendable detenerla, puesto que cuando levantemos nuestro docker nos dirá que están en uso, igualmente más tarde explicaremos como cambiar el puerto interno y externo dentro de docker.

Para evitar problemas es recomendable tener instalados los siguientes paquetes:
```bash
sudo apt-get update
sudo apt-get install apt-transport-https
sudo apt-get install curl
sudo apt-get install software-properties-common
```

### Instalar docker en nuestro equipo ###
Crear archivo para repositorio de docker

```bash
vim /etc/apt/sources.list.d/docker.list
deb [arch=amd64] https://download.docker.com/linux/ubuntu artful stable
```

Añadir clave GPG para Docker:
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Una vez importado actualizar paquetes disponibles
```bash
sudo apt-get update
```

### Instalar la versión CE de Docker ###
```bash
sudo apt-get install docker-ce
docker --version
```

### Evitar tener que escribir sudo cada vez que se escribe un comando de docker ###
```bash
sudo usermod -aG docker $USER
```

Ahora cerramos la sesión y volvemos a entrar, ya no debe ser necesario escribir sudo

### Instalar docker-compose ###
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

### Crear carpeta para almacenar los proyectos ###
```bash
cd /home/ajjimenez
mkdir docker
```

### Clonar el repositorio: ###
```bash
cd /home/ajjimenez/docker
git clone https://github.com/n0ni0/docker-lamp.git
cd docker-lamp
```

Una vez clonado el repositorio tendremos la siguiente estructura de carpetas:
![Docker]({{site.baseurl}}/assets/img/entornos-docker-lamp/directorio.png)

- apache En esta carpeta tendremos la configuración de nuestro servidor apache.
- hosts: Incluiremos los virtualhosts que vayamos creando
- db: Se almacenarán las bases de datos de todos nuestros proyectos
- fpm: Carpeta donde podremos modificar algunos parámetros de nuestra imagen de php
- public: Carpeta raíz para colocar todos nuestros proyectos
- docker-compose.yml: Archivo de configuración de nuestro entorno

### Configurar un proyecto dentro de nuestro docker ###
Ahora realizaremos los cambios necesarios para tener corriendo un wordpress dentro de nuestro docker sin realizar ningún cambio de versiones en el stack lamp.

- Crear el nuevo virtualhost para nuestro proyecto entrando en la carpeta apache/hosts

- Copiar y renombrar el archivo default.conf con el nombre que queramos, pero con extensión .conf
Importante tener en cuenta que si modificamos ó creamos un nuevo virtualhost, tendremos que reiniciar el contenedor de apache para que los cambios tengan efecto. Los comandos necesarios están en el README del [repositorio de github](https://github.com/n0ni0/docker-lamp){:target="_blank"}, no los añadiremos para no hacer demasiado extenso este tutorial.

- Para una configuración básica solo tendremos que modificar las líneas 3, 6 y 14 añadiéndole el nombre de la carpeta raíz que tendrá nuestro proyecto, en nuestro caso lo llamaremos wordpress. También modificaremos la línea 2 para agregar el server name – develop.wordpress.local

```
<VirtualHost *:80>
        ServerName develop.wordpress.local
        DocumentRoot /var/www/html/wordpress
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        Directory /var/www/html/wordpress;
            # enable the .htaccess rewrites
            Options Indexes FollowSymLinks MultiViews
            AllowOverride All
            Order allow,deny
            allow from all
            require all granted
       </Directory>
        ProxyPassMatch ^/(.*\.php(/.*)?)$ fcgi://phpfpm:9000/var/www/html/wordpress/$1
</VirtualHost>
```

Modificar el virtualhost de nuestro equipo para añadir el dominio de nuestro proyecto
```bash
sudo vim/etc/hosts
```

En nuestro caso para acceder al proyecto desde el navegador escribiremos develop.wordpress.local
```bash
127.0.0.1 develop.wordpress.local
```

- Copiar ó clonar nuestro proyecto dentro de la carpeta public, en nuestro caso se llamará wordpress

- Situarnos dentro de la carpeta raíz de nuestro docker y ejecutar el siguiente comando para descargarnos ó generar todas las imágenes necesarias y luego crear los contenedores.
```bash
docker-compose up -d
```

Una vez que lancemos el comando tardará un poco pués tendrá que crear nuestras imágenes customizadas de apache y php.

Si accedemos a nombre_de_nuestro_dominio:8080 tendremos acceso a un phpmyadmin donde podremos realizar las operaciones necesarias con las bases de datos

Una vez realizamos estos pasos si entramos a nombre_de_nuestro_dominio en nuestro navegador tendremos el asistente de instalación de wordpress.

Ahora vamos a explicar un poco la estructura de nuestro docker-compose.yml, pués podría ser necesario que cambiáramos algunos parámetros, igualmente es recomendable mirar en la documentación oficial como funciona si necesitamos realizar cambios.

```
version: "2"
services:
  db:
    image: 'mysql:5.6'
    volumes:
      - "./db/data:/var/lib/mysql"
      - "./db/init:/docker-entrypoint-initdb.d"
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: test
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    ports:
      - "3306:3306"
 
  mailcatcher:
    image: yappabe/mailcatcher
    ports:
        - "1025:1025"
        - "1080:1080"
    restart: unless-stopped
 
  php:
    depends_on:
      - mailcatcher
    build: fpm/
    links:
      - db:mysqldb
    volumes:
      - ./public:/var/www/html
      - ./fpm/custom.ini:/usr/local/etc/php/conf.d/custom.ini
    restart: unless-stopped
    ports:
      - "35729:35729"
 
  apache:
    build: apache/
    links:
      - php:phpfpm
    volumes:
      - ./public:/var/www/html
      - ./apache/hosts:/etc/apache2/sites-enabled
    restart: unless-stopped
    ports:
      - 80:80
 
  phpmyadmin:
    depends_on:
      - db
    image: phpmyadmin/phpmyadmin
    environment:
     - PMA_ARBITRARY=1
    restart: unless-stopped
    ports:
     - 8080:80
    volumes:
     - /sessions
```

db: Nombre interno que tendrá nuestro contenedor de bases de datos, los contenedores para comunicarse entre ellos usarán este nombre, pero si queremos acceder externamente a nuestra base de datos usaremos la ip 127.0.0.1

image: Nombre de la imagen de mysl que usaremos, si necesitamos usar otra versión la cambiaremos en este apartado, tendremos que realizar el comando docker-compose up -d para volver a reconfigurar nuestro entorno

volumes: Carpeta en nuestra raíz de docker / carpeta dentro del servidor. Estarán nuestras bases de datos ó algún script sql ó sh

restart: unless-stopped:  Evitaremos que el contenedor se reinicie cuando apaguemos docker

environment: Variables con las contraseñas de nuestro mysql

ports: Puerto del host / puerto del contenedor. Si tenemos conflicto con el puerto de mysql en nuestro equipo aquí sera donde los cambiaremos
Contenedor de PHP

```
php:
  depends_on:
    - mailcatcher
  build: fpm/
  links:
    - db:mysqldb
  volumes:
    - ./public:/var/www/html
    - ./fpm/custom.ini:/usr/local/etc/php/conf.d/custom.ini
  restart: unless-stopped
  ports:
    - "35729:35729"
```

depends_on: El contenedor de php tiene dependencias con el de mailcatcher

build: Ruta donde está el archivo Dockerfile en el que creamos nuestra imagen de php customizada, si queremos modificar algo en nuestro php tendremos que ir a esa ruta en la raíz de nuestro docker. Importante, si realizamos algún cambio, tendremos que borar el contenedor y imagen de php antes de volver a ejecutar de nuevo “docker-compose up -d”

links: Enlace a contenedores en otro servicio

Como último punto vamos a explicar un poco como hemos creado y como podemos modificar nuestra imágen customizada de php.

En el directorio fpm tenemos dos archivos:

Dockerfile:

```
FROM php:7.1-fpm
RUN apt-get update
     apt-get install -y \
     sudo \
     git \
     bzip2 \
     cron \
     wget \
     libfreetype6-dev \
     libjpeg62-turbo-dev \
     libmcrypt-dev \
     libpng-dev \
     libxml2-dev \
     libcurl4-openssl-dev \
     libxslt-dev \
     libicu-dev \
     nano \
     vim \
     gnupg \
    & docker-php-ext-install -j$(nproc) iconv mcrypt curl dom hash pdo pdo_mysql mysqli simplexml soap xsl intl zip opcache \
    & docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    & docker-php-ext-install -j$(nproc) gd
 
# ssmtp
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y ssmtp
 
# mysql-client
RUN apt-get update apt-get install -y mysql-client & rm -rf /var/lib/apt
 
# Xdebug
RUN pecl install xdebug-2.5.0
RUN docker-php-ext-enable xdebug
RUN sed -i '1 a xdebug.remote_autostart=true' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN sed -i '1 a xdebug.remote_mode=req' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN sed -i '1 a xdebug.remote_handler=dbgp' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN sed -i '1 a xdebug.remote_connect_back=1' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN sed -i '1 a xdebug.remote_port=9009' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN sed -i '1 a xdebug.remote_host=127.0.0.1' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN sed -i '1 a xdebug.remote_connect_back=1' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN sed -i '1 a xdebug.' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN sed -i '1 a xdebug.remote_enable=1' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
 
# Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
 
# Node.js
RUN curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
RUN apt-get install -y nodejs
 
# Gulp
RUN npm install --global gulp
```

Si deseamos cambiar la versión de php tendremos que modificar
FROM php:VERSION_DE_PHP-fpm

Si deseamos agregar algún paquete a nuestra imagen lo agregaremos con la instrucción RUN.

Tenemos instalado y configurado xdebug en el puerto 9009

custom.ini:
```
sendmail_path = /usr/sbin/ssmtp -t
memory_limit = 2048M
max_input_vars = 50000
max_execution_time = 0
max_input_nesting_level = 10000
max_input_time = -1
max_execution_time = 0
max_file_uploads = 1024M
upload_max_filesize = 1024M
post_max_size = 1025M
output_buffering = 4096
realpath_cache_size = 1M
realpath_cache_ttl = 86400
 
; Opcache
opcache.enable=1
opcache.revalidate_freq=0
opcache.max_accelerated_files=7963
opcache.memory_consumption=192
opcache.interned_strings_buffer=16
opcache.fast_shutdown=1
 
;Timezone
date.timezone = "Europe/Madrid"
```

En este archivo agregaremos las modificaciones que necesitemos al archivo php.ini

### Comandos básicos de docker que necesitaremos saber: ###
```
Mostrar todas las imágenes: # docker images
Mostrar los contenedores activos:  # docker ps
Mostrar todos los contenedores activos y detenidos:  # docker ps -a
Iniciar un contenedor: # docker start <nombre_contenedor>
Reiniciar un contenedor: # docker restart <nombre_contenedor>
Parar un contenedor: # docker stop <nombre_contenedor>
Borrar un contenedor (tiene que estar detenido para poder borrarlo): # docker rm <nombre del contenedor>
Acceder a un contenedor por consola: # docker exec -it <nombre_contenedor> bash
```

Ha sido un tutorial muy reducido para todas las configuraciones y modificaciones que se pueden realizar con docker, pero espero que sea de utilidad y como punto de partida para empezar a utilizar docker como entorno de desarrollo local.



