---
layout: post
title: Crear un módulo en Magento 2
date: 06/10/2018
description: Vamos a empezar una nueva sección dedicada a Magento 2, tratando temas tanto de frontend como de backend. En esta ocasión vamos a crear un módulo básico que nos muestre por pantalla la frase “Hello World”. # Add post description (optional)
img: modulo-m2/module_m2.png # Add image post (optional)
tags: [Magento2, Tutoriales] # add tag
---

Vamos a empezar una nueva sección dedicada a Magento 2, tratando temas tanto de frontend como de backend.
En esta ocasión vamos a crear un módulo básico que nos muestre por pantalla la frase “Hello World”.

### Crear la carpeta de nuestro módulo ###
Los modulos los crearemos dentro de la carpeta app/code.
El nombre de las carpetas donde tendremos nuestro módulo será en base a la nomenclatura “VendorName_ModuleName”
VendorName:  Suele ser el nombre de nuestra empresa ó  nuestro nombre.
ModuleName: Es el nombre que le daremos al módulo.

En nuestro caso creamos las siguientes carpetas:
```bash
app/code/Ajimenez/Helloworld
```

### Crear el archivo registration.php ###
Este archivo es el encargado de registrar nuestro módulo en el proyecto, es muy importante que lo escribamos bien, si nos confundimos en una letra ó mayúscula no funcionará.

```php
<?php
\Magento\Framework\Component\ComponentRegistrar::register(
    \Magento\Framework\Component\ComponentRegistrar::MODULE,
    'Ajimenez_Helloworld',
    __DIR__
);
```

### Crear el archivo module.xml ###
Tendremos que crear una carpeta etc en la raíz de nuestro módulo y dentro de ella crear el archivo module.xml, la función de este archivo es decirle a magento la versión actual del módulo.
Si le añadiéramos el parámetro <sequence> le diríamos en el orden que se debe de instalar el módulo y las dependencias.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/etc/module.xsd">
    <module name="Ajimenez_Helloworld" setup_version="0.0.1" />
</config>
```

Llegamos a este punto ya tenemos los archivos necesarios para poder activar nuestro módulo, eso sí, no tiene ninguna funcionalidad.

Si nos vamos a la raíz de nuestro proyecto y ejecutamos el siguiente comando podremos ver que nuestro módulo aparece, pero no está activado.

```bash
php bin/magento module:status
```

```
List of disabled modules:
Ajimenez_Helloworld
```

Para activarlo ejecutaremos:

```bash
php bin/magento setup:upgrade
```

Este comando activará nuestro módulo, y si tuvieramos algún archivo de instalación ó modificación de base de datos también se ejecutaría.

Si volvemos a ejecutar el comando para ver el estado de los módulos ya no nos aparecerá.
También lo podemos ver el estado de los modulos revisando en el archivo app/etc/config.php.

Si el módulo está activado tendra un 1, y si está desactivado un 0

```php
<?php
return [
    'modules' => [
        'Ajimenez_Helloworld' => 1,
```

### Crear el arhivo routes.xml ###
Este archivo es el encargado de crear la ruta y llamar al controlador correspondiente.

router:
Acepta dos parámetros, standard (para una ruta en la parte front de la web) y admin (para una ruta en el panel de gestión)

route:
id : Es el identificador único de la ruta
frontName: Url que llamará a nuestro controlador

Crearemos este archivo para la parte front de nuestra web por lo que lo crearemos en el siguiente directorio:

```
app/code/Ajimenez/Helloworld/etc/frontend/routes.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:App/etc/routes.xsd">
    <router id="standard">
        <route id="helloworld" frontName="helloworld">
            <module name="Ajimenez_Helloworld" />
        </route>
    </router>
</config>
```

### Crear el controlador ###
El controlador es el encargado de recibir las requests, procesarlas y renderizar la página.

Si la carpeta del controlador es Index y el controlador también se llama Index.php podemos omitirlos en el navegador, escribiendo https://nuestro_dominio/nombre_ruta

```
app/code/Ajimenez/Helloworld/Controller/Index/Index.php
```

```php
<?php
 
namespace Ajimenez\Helloworld\Controller\Index;
 
class Index extends \Magento\Framework\App\Action\Action
{
    protected $_pageFactory;
 
    public function __construct(
        \Magento\Framework\App\Action\Context $context,
        \Magento\Framework\View\Result\PageFactory $pageFactory)
    {
        $this->_pageFactory = $pageFactory;
        return parent::__construct($context);
    }
 
    public function execute()
    {
        return $this->_pageFactory->create();
    }
}
```

Todos los controladores deben de extender de la clase \Magento\Framework\App\Action\Action e implementar el método execute.

Los controladores solo pueden ejecutar una acción que estará en el método execute.
Podemos crear tantos métodos como queramos para desacoplar y hacer más manejable a execute.

En nuestro controlador hemos utilizado la injección de dependencias para crear una factoría, esta en concreto nos renderizará todo el frontend como en el resto de la web, en vez de mostrarnos una nueva página en blanco sin contenido.

### Crear el bloque, el layout y una plantilla ###
Los layouts son los ficheros que definen la estructura de una página. Nos permiten desde crear una estructura desde 0 a extender una existente, editar bloques de contenido llamándolos por su id, borrar bloques, … Tiene un gran potencial y es sumamente importante conocer su funcionamiento.

Los layouts pueden ser para la parte front, para la back ó para ambos.

En nuestro caso deben de ir en la carpeta:
```
Ajimenez/Helloworld/view/frontend/layout
```

Para crear un layout de una página en concreto, magento sigue la nomenclatura:
```
<frontName>/<controller_folder_name>/<controller_class_name>
```

En nuestro caso tendremos que crear:
```
app/code/Ajimenez/Helloworld/view/frontend/layout/helloworld_index_index.xml
```

```xml
<?xml version="1.0"?>
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" layout="1column" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <referenceContainer name="content">
        <block class="Ajimenez\Helloworld\Block\Index" name="helloworld" template="Ajimenez_Helloworld::index.phtml" />
    </referenceContainer>
</page>
```

Hacemos referencia al contenedor “content” que es el contenedor principal de todas la páginas y le añadimos un nuevo bloque llamado “helloworld” que usa la tempate index.phtml que crearemos acontinuación.

Ahora crearemos el bloque que será el encargado de proporcionar funcionalidades a nuestra plantilla.

```
app/code/Ajimenez/Helloworld/Block/Index.php
```

```php
<?php
 
namespace Ajimenez\Helloworld\Block;
 
 
class Index extends \Magento\Framework\View\Element\Template
{
    public function showHelloWorld()
    {
        return 'Hello world';
    }
}
```

Finalmente, creamos nuestra plantilla en la siguiente ruta:
```
app/code/Ajimenez/Helloworld/view/frontend/templates/index.phtml
```

```php
<?php echo $block->showHelloWorld(); ?>
```

Con $block tenemos acceso a los métodos del bloque que hemos creado, en este caso a showHelloWorld en nuestra plantilla renderizando “Hello world”

Si ahora nos vamos a nuestro navegador y escribimos:
```
https://my_dominio/helloworld
```

Veremos nuestra frase “Hello World” renderizada.

![Docker]({{site.baseurl}}/assets/img/modulo-m2/m2_module_screen.png)

Esto a sido todo, hemos tratado muchos puntos importantes de manera superficial pero espero que sirvan de punto de partida.


