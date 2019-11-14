# Principio de segregación de interfaz

## Ejercicios

Dado el siguiente interfaz de item IItem:

```php
<?php
interface IItem
{
    public function applyDiscount($discount);
    public function applyPromocode($promocode);

    public function setColor($color);
    public function setSize($size);
    public function setMaterial($material);
    
    public function setPrice($price);
}
?>
```

1) Programar una clase Book y una clase Shirt que implementen el interfaz
2) Crear un snippet de código que cree un nuevo libro y otro que cree una nueva camiseta aplicando todos los métodos del interfaz que sean aplicables a cada objeto.

¿Os gusta este interfaz? 

Este interfaz no es bueno, porque tiene demasiados métodos. Hay métodos que no tienen sentido para objetos Book y otros que no tienen sentido para objetos Shirt. Además ¿qué ocurre si algún tipo de Item no puede tener descuentos o códigos promocionales? El interfaz está obligando a implementar todos los métodos aunque no tengan sentido para algunos tipos de Item.

El principio de segregación de interfaz dice que es mejor tener varios interfaces con pocos métodos que un interfaz con muchos métodos.

3) Separar el interfaz IItem en varios interfaces
4) Reescribir las clases Book y Shirt según los nuevos interfaces

```php
<?php
interface IItem
{
    public function setCondition($condition);
    public function setPrice($price);
}

interface IClothes
{
    public function setColor($color);
    public function setSize($size);
    public function setMaterial($material);
}

interface IDiscountable
{
    public function applyDiscount($discount);
    public function applyPromocode($promocode);
}

class Book implemets IItem, IDiscountable
{
    public function setCondition($condition){/*...*/}
    public function setPrice($price){/*...*/}
    public function applyDiscount($discount){/*...*/}
    public function applyPromocode($promocode){/*...*/}
}

class KidsClothes implemets IItem, IClothes
{
    public function setCondition($condition){/*...*/}
    public function setPrice($price){/*...*/}
    public function setColor($color){/*...*/}
    public function setSize($size){/*...*/}
    public function setMaterial($material){/*...*/}
}
?>
```

## Ejercicios

### Ejercicio 1

Segregar el interfaz y crear las clases Car, Bus, Motorbike, Bike

```php
interface VehicleInterface {
    function startEngine();
    function accelerate();
    function brake();
    function lightsOn();
    function lightsOff();
    function signalLeft();
    function signalRight();
    function changeGear($gear);
    function startRadio();
    function stopRadio();
    function ejectCD();
    function openFrontDoor();
    function openBackDoor();
}
```

### Ejercicio 2

Arreglar el siguiente diseño de clases e interfaz:

```php
interface IAve {  
    function volar();
    function comer();
    function nadar();
}

class Loro implements IAve{

    public function volar() {
        //...
    }

    public function comer() {
        //...
    }

    public function nadar() {
        //...
    }
}

class Pinguino implements IAve{

    public function volar() {
        //...
    }

    public function comer() {
        //...
    }

    public function nadar() {
        //...
    }
}
```

## Anti patrones

- Fat interfaces