---
layout: post
title: Configurar xdebug en phpstorm en nuestro entorno docker
date: 06/10/2018
description: En este post vamos a configurar xdebug en el IDE PhpStorm usando el entorno docker que configuramos en el post anterior, entornos de desarrollo lamp con docker # Add post description (optional)
img: configurar-xdebug/configurar_xdebug.png # Add image post (optional)
tags: [Docker, Lamp, phpStorm, xdebug] # add tag
---

En este post vamos a configurar xdebug en el IDE [phpStorm](https://www.jetbrains.com/phpstorm/){:target="_blank"} usando el entorno docker que configuramos en el post anterior, [entornos de desarrollo lamp con docker](https://antoniojimenezvelazquez.es/tutoriales/entornos-de-desarrollo-lamp-con-docker/){:target="_blank"}

Nuestro entorno ya tiene habilitado y funcionando xdebug en el puerto 9009, por lo que no necesitamos tocar nada en los archivos de configuración de docker, solo tenemos que configurar PhpStorm para que escuche en nuestro puerto y decirle la ruta del proyecto.

### Cambiar el puerto de escucha de xdebug ###
Tenemos que cambiar el puerto de escucha al 9009, para ello abrimos las opciones del editor y nos vamos a **Languages & Frameworks / PHP / Debug**

![Docker]({{site.baseurl}}/assets/img/configurar-xdebug/phpstorm_puerto_escucha.png)

### Crear una nueva configuración para xdebug ###
Nos iremos en la barra de navegación superior a Run | Edit configurations …
En la columna de la izquierda pulsamos en el simbolo + y seleccionamos Php Web Page.
En el apartado name agregar un nombre, en nuestro caso le pondremos “myProyect”.

![Docker]({{site.baseurl}}/assets/img/configurar-xdebug/xdebug_config.png)

- Por último nos queda agregar la ruta de nuestro proyecto, para ello pulsamos en los 3 puntitos a la derecha de server.
Name: le damos un nombre, seguiremos dándole myProyect.
- Host: debe de ir la url de nuestro entorno local, ejemplo:  http://myproyect.local
- Marcamos el check que dice “Use path mappings”
- Hacemos click en la primera línea que nos aparece en la columna de la derecha “Absolute path on the server” y escribimos la ruta donde está nuestro proyecto dentro del contenedor de php-fpm de docker, la parte que nunca cambiará es /var/www/html/ la siguiente carpeta será donde está nuestro proyecto (esta carpeta es la que tenemos en la raíz de nuestro docker, public, donde dijimos que meteríamos todos nuestros proyectos.) En nuestro caso escribiremos /var/www/html/myproyect
- Pulsamos en ok para aceptar los cambios

![Docker]({{site.baseurl}}/assets/img/configurar-xdebug/phpstorm_puerto_escucha.png)

Se cierra la ventana del server y volvemos a la anterior.
Debemos de asegurarnos de que en el campo Start URL tengamos la url correcta de nuestro proyecto en local, http://myproyect.local
Pulsamos en Ok y se nos cierran todas las pantallas.

### Crear un punto de ruptura y testearlo ###
Hacer click a la izquierda de la línea y nos aparecerá un punto rojo a la izquierda.

![Docker]({{site.baseurl}}/assets/img/configurar-xdebug/punto_ruptura.png)

Para activar xdebug hay que hacer click en el icono que tenemos a la derecha del play arriba en la barra de navegación

![Docker]({{site.baseurl}}/assets/img/configurar-xdebug/punto_ruptura_icon.png)

Con esto podemos dar por finalizada la configuración de xdebug usando docker. Aclarar que hay muchas maneras de configurar xdebug con PhpStorm, yo he posteado la que yo uso 😀
No hemos entrado en el tema de como se usa xdebug porque no es el objetivo de este post.