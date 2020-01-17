---
layout: post
title: Inyección de dependencias en Magento 2
date: 05/11/2018
description: ¿Qué es la Inyección de dependencias? La inyección de dependencias es un patrón de diseño orientado a objetos en el que se suministran objetos a una clase en lugar de ser la propia clase la que cree dichos objetos. # Add post description (optional)
img: di-m2/di_m2.png # Add image post (optional)
tags: [Magento2, Tutoriales] # add tag
---

### ¿Qué es la Inyección de dependencias? ###
La inyección de dependencias es un patrón de diseño orientado a objetos en el que se suministran objetos a una clase en lugar de ser la propia clase la que cree dichos objetos.

Entre las principales ventajas cabe destacar:

Permite que nuestro código sea más flexible ya que hace que una clase pueda cambiar sus dependencias sin necesidad de cambiar su código.
Vemos de forma más rápida y ordenada todas las dependencias al tenerlas todas en nuestro constructor y no desperdigadas por el resto del código.
Dejamos de llamar a cada clase que necesitemos con “new” y usamos objetos ó factorías en el constructor.
Cuando creamos un objeto ya tenemos todas las dependencias que necesitamos.
En Magento 1 los objetos eran construidos con la clase “Mage”, en Magento 2 usamos la inyección de dependencias y el Object Manager, consiguiendo un desacoplo del código entre otras muchas mejoras.

Magento 2 usa dos tipos de inyección de dependencias la inyección por constructor y la inyección por métodos.

Usaremos el mismo ejemplo que encontramos en la documentación oficial:

```php
namespace Magento\Backend\Model\Menu;
class Builder
{
    /**
     * @param \Magento\Backend\Model\Menu\Item\Factory $menuItemFactory
     * @param \Magento\Backend\Model\Menu $menu
     */
    public function __construct(
        Magento\Backend\Model\Menu\Item\Factory $menuItemFactory,  // Service dependency
        Magento\Backend\Model\Menu $menu  // Service dependency
    ) {
        $this->_itemFactory = $menuItemFactory;
        $this->_menu = $menu;
    }
 
    public function processCommand(\Magento\Backend\Model\Menu\Builder\CommandAbstract $command) // API param
    {
        // processCommand Code
    }
}
```

En el ejemplo podemos observar como la clase Builder declara sus dependencias para menuFactory y para menu usando la inyección por constructor. Es el método más común.

Para el método processComand pasa la inyección de dependencias como parámetro. Este tipo de inyección se suele usar cuando la dependencia puede variar en cada llamada al método.

### ¿Qué es Object Manager y para que sirve? ###
El Object Manager como la mayoría de las clases en Magento 2, implementa una interfaz, en este caso Magento\Framework\ObjectManagerInterface.

Object Manager es la clase hace funcionar la inyección de dependencias automática en Magento 2.
Crea una instancia nueva de los objetos que le pasamos e implementa el patrón de diseño singleton, controla la creación de instancias de parámetros (incluidos en todos los di.xml del proyecto) y también modifica la instancia de objetos, puesto que usando una preference le podemos decir a magento que clase usar para instanciar un objeto.

Todas las dependencias de una clase son listadas como argumentos en el constructor. Cuando magento necesita una instancia de una clase ó una interfaz le pide al Object Manager que la cree.
En resumidas cuentas, el Object Manager es el contenedor de dependencias de Magento 2.

Crea una instancia de las dependencias para un nuevo objeto y se asegura de que cada objeto se cree solo una única vez.

Lo más importante de todo, no debemos llamar al Object Manager directamente, debemos de usar la inyección de dependencias. Magento en sus buenas prácticas nos lo dice claramente. Si revisamos al core de Magento podemos ver llamadas directamente al Object Manager, como también nos dicen en la documentación, no debe de usarse como ejemplo y solo existen para compatibilidad con versiones anteriores por lo que en un futuro desaparecerán.

Para recopilar todas las dependencias que necesitamos inyectar en el Object Manager usamos los archivos di.xml que tenemos en cada módulo. Podemos tener un di.xml para cada área permitiendo configuraciones diferentes según el área.

### Objetos inyectables y no inyectables ###
En Magento 2 tenemos dos grupos de objetos, lo que podemos inyectar y los que no.

Objetos no inyectables:
Son objetos que tienen una entidad, como por ejemplo los productos, usuarios, pedidos, …
Si necesitamos usar un objeto de este tipo no los podremos llamar usando Object Manager, tendremos que llamar a una factoría.

Objetos inyectables:
Pueden ser llamados como servicios o objetos y muestran sus dependencias en sus constructores y son creados por el Object Manager ó usando la configuración por archivos di.xml.

### ¿Cómo localizar las dependencias que necesito? ###
Una manera sería mirar en el directorio API del módulo que vamos a llamar en nuestro constructor. Las clases que encontremos en esta carpeta han sido creadas específicamente para llamarlas desde otros módulos y suelen ser más estables que el resto de clases del módulo.
Con el tiempo a base de llamar a las dependencias cada vez que creamos un constructor iremos sabiendo a cuales tenemos que llamar, esto nos pasa con cualquier CMS o framework que empecemos a usar, no hay por qué preocuparse, solo tener paciencia y siempre tendremos a nuestro amigo stack overflow 🙂

Como última recomendación para cuando estemos desarrollando y empecemos a ver que nuestro constructor se está haciendo sumamente enorme es hora de pararnos, ver qué podríamos sacar a otra clase e intentar refactorizarlo, nos puede ahorrar bastantes quebraderos de cabeza.

