---
layout: post
title: View Models en Magento 2
date: 16/03/2019
description: Para crear una vista en Magento 2 necesitamos un layout, un bloque y un template. # Add post description (optional)
img: view-model/viewmodel_m2.png # Add image post (optional)
tags: [Magento2, Tutoriales] # add tag
---

Para crear una vista en Magento 2 necesitamos un layout, un bloque y un template.

Layout:

Es un archivo *.xml en el que tendremos los contenedores y bloques, pudiendo ser uno nuevo ó extender uno existente.


En él crearemos la estructura de la página.

Bloque:

Es una clase php que se usa para conectar el layout con el template.

En ella debemos de agregar la lógica de negocio que le queremos pasar a nuestro template.

Template:

Archivo *.phtml. En ese archivo no se debe de incluir la lógica de negocio.

Lo usaremos para llamar a los métodos de nuestro bloque, métodos que nos proporciona por defecto magento y código html ó js para maquetar la vista.

Anteriormente a la versión 2.2, los bloques eran la manera recomendada de pasar la lógica de negocio a nuestros templates.

A partir de la versión 2.2, los view models son la manera recomendada, entre sus ventajas podemos destacar:

Tenemos un constructor mucho más ligero, solo inyectaremos nuestras dependencias.
En caso de ser necesario extender nuestra clase ya no tendremos que preocuparnos de las dependencias del constructor padre, con el consiguiente aumento del tamaño de nuestro constructor.
Otro cambio importante en la versión 2.2 es que los bloques ya se consideran de forma automática de tipo “Template”, por lo que no es necesario especificar el tipo en el layout.

Una vez hecha esta pequeña introducción, vamos a mostrar un ejemplo de como crear un view model.

Crearemos un modulo que agregará una nueva pestaña llamada “custom” en la ficha del producto, y en ella mostraremos los 3 primeros productos con un precio superior a 91€.

Solo vamos a mostrar los archivos importantes, si queremos ver el código completo lo podremos ver en [github](https://github.com/n0ni0/ProductCollection){:target="_blank"}

![Docker]({{site.baseurl}}/assets/img/view-model/module_structure.png)

```xml
<?xml version="1.0"?>
<page layout="1column" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
        <referenceBlock name="product.info.details">
            <block  name="demo.tab" template="Ajjimenez_ProductCollection::custom_tab.phtml" group="detailed_info">
                <arguments>
                    <argument translate="true" name="title" xsi:type="string">Custom Tab</argument>
                    <argument name="view_model" xsi:type="object">Ajjimenez\ProductCollection\ViewModel\Product</argument>
                </arguments>
            </block>
        </referenceBlock>
    </body>
</page>
```

En el código anterior modificamos el layout de la página de producto.

Pasamos como argumento al bloque el view model, de tipo objeto y especificamos la ruta donde irá nuestra clase

```php
<?php
 
namespace Ajjimenez\ProductCollection\ViewModel;
 
 
class Product implements \Magento\Framework\View\Element\Block\ArgumentInterface
{
    /**
     * @var \Magento\Catalog\Model\ResourceModel\Product\CollectionFactory
     */
    private $collectionFactory;
 
    /**
     * @var \Magento\Store\Model\StoreManagerInterface
     */
    private $storeManager;
 
    /**
     * Product constructor.
     * @param \Magento\Catalog\Model\ResourceModel\Product\CollectionFactory $collectionFactory
     * @param \Magento\Store\Model\StoreManagerInterface $storeManager
     */
    public function __construct(
        \Magento\Catalog\Model\ResourceModel\Product\CollectionFactory $collectionFactory,
        \Magento\Store\Model\StoreManagerInterface $storeManager
    ) {
        $this->collectionFactory = $collectionFactory;
        $this->storeManager = $storeManager;
    }
 
    /**
     * @return \Magento\Catalog\Model\ResourceModel\Product\Collection
     */
    public function getFilteredCollection()
    {
        $collection = $this->collectionFactory->create();
        $collection->addFieldToSelect('*');
        $collection->addAttributeToFilter('price', ['gt' => 91]);
        $collection->setOrder('name', 'ASC');
        $collection->setPageSize(3);
 
        return $collection;
    }
 
    /**
     * @param $product
     * @return string
     * @throws \Magento\Framework\Exception\NoSuchEntityException
     */
    public function getProductImage($product)
    {
        return $this->storeManager->getStore()->getBaseUrl(\Magento\Framework\UrlInterface::URL_TYPE_MEDIA) . 'catalog/product' . $product->getImage();
    }
 
}
```

La clase Product implementa la clase ArgumentInterface.

En el constructor solo inyectamos las dependencias que vamos a usar nosotos, ninguna que necesite magento.

Creamos los dos métodos que necesitamos.

```php
<?php // Get current product
 
$viewModel = $block->getViewModel();
foreach ($viewModel->getFilteredCollection() as $product) {
    echo $product->getName() . '<img height="100" width="150" src="'. $viewModel->getProductImage($product) . '"/></br>';
}
?>
```

Finalmente creamos la plantilla.

Para acceder a los métodos que creamos en nuestro view model, guardamos en la variable $viewModel $block->getViewModel();

Ya podemos acceder a los métodos usando $viewModel->mi_método().