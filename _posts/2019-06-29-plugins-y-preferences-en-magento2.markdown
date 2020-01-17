---
layout: post
title: Plugins y preferences en magento 2
date: 29/06/2019
description: Una de las tareas más comunes es modificar clases y métodos en los módulos del core ó de un módulo de terceros. # Add post description (optional)
img: plugins-preferences-m2/plugins_preferences_m2.png # Add image post (optional)
tags: [Magento2, Tutoriales] # add tag
---

Una de las tareas más comunes es modificar clases y métodos en los módulos del core ó de un módulo de terceros.

Hay dos maneras de realizar estos cambios, usando plugins ó preferences. Veamos cada una de ellas.

### Plugins: ###
Los plugins nos permiten realizar modificaciones en un método sin modificar el código original.

Cuando usamos un plugin, se crea automáticamente una clase ‘interceptor’ que incluye todos los métodos que hemos creado en nuestra clase plugin. Esta clase es la que finalmente se ejecuta en vez de la original.

Cada vez que se realice una modificación en nuestra clase plugin, tendremos que borrar el interceptor autogenerado en la carpeta /generated.

Los plugins tienen algunas limitaciones, no podremos crearlos en:

Métodos que no sean públicos
Clases finales
Métodos finales
Métodos de clases
Función __construct
Virtual types
Objetos instanciados antes de que Magento\Framework\Interception sea lanzado
Pueden ser usados en interfaces, clases abstractas y clases padre

Deben de ir siempre dentro de un módulo en la carpeta app/code.

Para la declaración de un plugin es necesario el archivo di.xml este archivo es usado para la inyección de dependencias, su ubicación variará según él área donde ejecutemos nuestro plugin, quedando de la siguiente manera: /etc/frontend/<area>/di.xml.

```xml
<?xml version="1.0" ?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <type name="{ObservedType}">
       <plugin name="{pluginName}" type="{PluginClassName}" sortOrder="1" disabled="false" />
    </type>
</config>
```

Parámetros obligatorios a la hora de crear un plugin:

- type name: Clase o interfaz que el plugin va a observar.
- plugin name: Nombre con el que se identificará el plugin.
- plugin type: Nombre de la clase ó virtual type donde implementamos el plugin

Parámetros no obligatorios:
- sortOrder: Orden en el que se ejecutará el plugin en caso de que existan más plugins.
- disabled: Desactiva el plugin con el valor true, por defecto tiene el valor false.

Los plugins pueden ser de 3 tipos:
- Before: Ejecutan el código antes de llamar al método original.
- After: Ejecutan código después de llamar al método original.
- Around: Ejecutan código antes y después de llamar al método original.


#### Before ####
- Permite realizar modificaciones antes de la llamada al método original y cambiar los argumentos pasados.
- Los métodos a sobreescribir deben de tener el nombre original del método con el prefijo ‘before’.
- Recibe como parámetro $subject (objeto original), y luego todos los argumentos que recibe el método original.
- Si el método tiene más de un argumento, debe devolverse como un array.

```php
<?php
namespace My\Module\Plugin;
 
class ProductAttributesUpdater
{
    public function beforeSetName(\Magento\Catalog\Model\Product $subject, $name)
    
        return ['(' . $name . ')'];
    
}
```

#### After ####
- Permite realizar modificaciones después de la llamada al método original.
- Los métodos a sobreescribir deben de tener el nombre original del método con el prefijo after.
- Recibe como parámetros $subject (objeto original) y $result, que es el resultado del método original.
- No necesitan declarar todos los parámetros del método original, excepto aquellos que sean necesarios en el plugin.

```php
<?php
 
namespace My\Module\Plugin;
 
class ProductAttributesUpdater
{
    public function afterGetName(\Magento\Catalog\Model\Product $subject, $result)
    
        return '|' . $result . '|';
    
}
```

#### Around ####
- Permite realizar modificaciones mientras se ejecuta el método original, es posible cambiar por completo la funcionalidad.
- Los métodos a sobreescribir deben de tener el nombre original del método con el prefijo around.
- Recibe como parámetros $subject (objecto original), $proceed (función original del método)
- Siguiendo las recomendaciones de la documentación, se deben de usar como última opción ya que aumentan la complejidad del código, localización de errores e incluso pueden llegar a afectar en el rendimiento.

```php
<?php
namespace My\Module\Plugin;
 
class ProductAttributesUpdater
{
    public function aroundSave(\Magento\Catalog\Model\Product $subject, callable $proceed)
    
        $someValue = $this->doSmthBeforeProductIsSaved();
        $returnValue = null;
        
        if ($this->canCallProceedCallable($someValue)) {
            $returnValue = $proceed();
        
        
        if ($returnValue) {
            $this->postProductToFacebook();
        
        
        return $returnValue;
    
}
```

### Preferences: ###
Una preference nos permite realizar modificaciones en una clase sin necesidad de modificar la clase original, ya sea tanto una clase del core como de un módulo de terceros.

Estos cambios son más propensos a errores cuando otros módulos llamen a la clase modificada ó cuando se actualice la versión de magento.

La declaración de una preference es similar a la de un plugin.

Deberemos de crear el archivo di.xml, este archivo es usado para la inyección de dependencias, su ubicación variará según él área donde ejecutemos nuestro plugin, quedando de la siguiente manera: /etc/frontend/<area>/di.xml.

```xml
<?xml version="1.0" ?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
        <preference for="Magento\Catalog\Api\Data\ProductInterface" type="Magento\Catalog\Model\Product" />
</config>
```

Siempre que sea posible, debemos de llamar a la interfaz de la clase sobre la que queremos hacer la modificación.

Parámetros obligatorios a la hora de crear una preference:

- for: Nombre de la interfaz original sobre la cual vamos a realizar los cambios.
- type: Nombre de la clase que va a sobreescribir a la original
La clase que creamos debe de extender de la clase original y solo implementar el ó los métodos que necesitan modificaciones.

Crearemos en nuestro módulo la clase a sobreescribir siguiendo la misma estructura de carpetas que tenga en el módulo original para evitar posibles confusiones.

```php
namespace MyModule\Catalog\Model;
 
class Product extends \Magento\Catalog\Model\Product
{
// Implementación solo los métodos que necesitemos modificar
}
```

Una vez vistas las dos formas de sobreescribir funcionalidades en magento, solo hay que elegir la que más se ajuste a nuestro desarrollo.

- Los plugins los usaremos para añadir funcionalidades antes, durante y después de la ejecución del método.
- Las preferences las usaremos cuando no sea posible usar un plugin ó para sobrescribir una clase.
