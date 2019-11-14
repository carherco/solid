# Prinicipio de sustitución de Liskov

"Child classes should never break the parent class' type definitions."

Esto es fácilmente comprobable de la siguiente manera:

"Subtypes must be substitutable for their base types."

## Ejemplo

Clase padre: 

```php
class Vehicle {
 
    function startEngine() {
        // Default engine start functionality
    }
 
    function accelerate() {
        // Default acceleration functionality
    }
}
```

Clases hijas:

```php
class Car extends Vehicle {
 
    function startEngine() {
        $this->engageIgnition();
        parent::startEngine();
    }
 
    private function engageIgnition() {
        // Ignition procedure
    }
 
}
 
class ElectricBus extends Vehicle {
 
    function accelerate() {
        $this->increaseVoltage();
        $this->connectIndividualEngines();
    }
 
    private function increaseVoltage() {
        // Electric logic
    }
 
    private function connectIndividualEngines() {
        // Connection logic
    }
 
}
```

Uso:

```php
class Driver {
    function go(Vehicle $v) {
        $v->startEngine();
        $v->accelerate();
    }
}
```

Nuestro Driver es capaz de utilizar cualquiera de los vehículos, incluso utilizar el objeto padre Vehicle.


## Ejemplo de violación de LSP

```php
<?php
class Rectangle
{
    protected $width;
    protected $height;

    public setWidth($width)
    {
        $this->width = $width;
    }

    public setHeight($height)
    {
        $this->height = $height;
    }

    public function getWidth()
    {
        return $this->width;
    }

    public function getHeight()
    {
        return $this->height;
    }
}

class Square extends Rectangle
{
    public setWidth($width)
    {
        parent::setWidth($width);
        parent::setHeight($width);
    }

    public setHeight($height)
    {
        parent::setHeight($height);
        parent::setWidth($height);
    }
}

/** Calcula el área pasándole el tipo de figura y el ancho y el alto */
function calculateRectangleArea(Rectangle $rectangle, $width, $height)
{
    $rectangle->setWidth($width);
    $rectangle->setHeight($height);
    return $rectangle->getHeight * $rectangle->getWidth;
}

calculateRectangleArea(new Rectangle(), 4, 5); // 20

/** Sustituimos una clase (Rectangle) por la otra (Square), sin modificar el resto del código **/
calculateRectangleArea(new Square(), 4, 5); // 25 ???
```

El resultado (25) no es correcto. Pero ya antes de obtener el resultado algo no nos cuadraba: Al cambiar Rectangle por Square, nos sobraba un parámetro para la función calculateRectangleArea().

¿Cuál es el problema? ¿Es que acaso un cuadrado no es un rectángulo? En términos geométricos sí, pero no en términos de Programación Orientada a Objetos.

Cuando tengamos tests, si un tests hecho para la clase padre no funciona para la clase hija, hemos roto el principio de sustitución de Liskov.

```php
public function testArea() {
    $r = new Rectangle();
    $r->setWidth(4);
    $r->setHeight(5);
    assertEquals(20, $r->calculateArea());
}
```

Una clase hija debe respetar el "contrato" de la clase padre. Si no lo respeta, no puede ser hija suya. Y en el caso de Rectangle, el comportamiento es que se necesitan establecer por separado un width y un height. Y la clase Square no respeta eso.

Además de respetar los "contratos", se deben respetar las siguientes normas:

- Preconditions can not be strengthened in a subclass.

Asumamos que nuestra clase base trabaja con una propiedad que es un int. Ahora, creamos una subclase que requiere (restrige) esa propiedad int para que sea un entero positivo. Esto es reforzar (hacer más restrictiva) las precondiciones. Cualquiero código que funcionara perfectamente con números negativos dejará de funcionar con la subclase.

- Postconditions can not be weakened in a subclass.

Asumamos el mismo escenario, pero en este caso la clase base garantiza que la propiedad sea positiva (no acepta números que no sean positivos en su setter). Y en este caso, la clase base sobreescribe el comportamiento del setter para que permita números negativos. Un código que use un getter de la propiedad asumiendo que siempre va a ser positivo (porque la clase base lo garantizaba) ahora estará roto debido a que se ha debilitado dicha restricción del getter.

A veces encontraremos solución para que las clases hijas respeten los contratos:

- cambiando el comportamiento de las clases hijas al contrato de la clase padre o del interfaz
- cambiando el contrato (es decir, cambiando cómo se van a utilizar las clases desde fuera)

En otras ocasiones, terminaremos dándonos cuenta de que realmente tenemos contratos distintos, y por lo tanto, las clases no deben estar relacionadas entre ellas.

## Ejercicios

### Ejercicio 1

```php
interface UserRepository {
    /** 
     * @return collection
     */
    public function getUserData($userId);
}

class NormalUserRepository implements UserRepository {
    /** returns a collection containing user data */
    public function getUserData($userId) {
        return DB::Table('users')->where('user_id', '=', $userId);
    }
}

class FilesystemUserRepository implements UserRepository {
    /** returns an array of users */
    public function getUserData($userId) {
        return filesystem::getUserInformation($userId)
    }
}
```

En este ejercicio, la clase FilesystemUserRepository "rompe" el contrato establecido por el interfaz UserRepository.

