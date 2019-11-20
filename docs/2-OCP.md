# Principio Open-Closed

Las clases que usas deberían estar:
- abiertas para poder extenderse
- cerradas para modificarse.

Traducción: deben poder extenderse sin modificarlas.

## Ejemplos

### Ejemplo 1

```php
class LoginModule {
    
    public function login($user) {
        if($user instanceof NormalUser) {
            $this->authenticateNormalUser($user);
        } else if($user instanceof GmailUser) {    
            $this->authenticateGmailUser($user);
        }
    }

    public function authenticateNormalUser($user) {
        // ...
    }

    public function authenticateGmailUser($user) {
        // ...
    }
}
```

¿Y si queremos añadir más tipos de usuario con sus métodos de autenticación?

Solución

1) Aplicar principio SRP:

```php
interface AuthenticateUserInterface {
    public function authenticate($user);
}

class NormalUserAuthenticator implements AuthenticateUserInterface {
    public function authenticate($user) {
        // ...
    }
}

class GmailUserAuthenticator implements AuthenticateUserInterface {
    public function authenticate($user) {
        // ...
    }
}
```

2) Aplicar el patrón *strategy*:

```php
class LoginModule {
    
    public function login($user) {
        $class = get_class($user).'Authenticator';
        $class::authenticate($user);
    }
}
```

La clase LoginModule ha quedado completamente CERRADA. No hace falta modificarla para meter más métodos de autenticación.

Esta solución está basada en el **strategy pattern**. También suele encajar muy bien en estos casos el **factory pattern**.

### Ejemplo 2

En este caso, tenemos una clase Repository que se conecta a base de datos (podría ser MySQL, por ejemplo). Pero de repente queremos trabajar con una API en vez de con la base de datos.

```php
<?php
class OrderRepository
{
    public function find($orderID)
    {
        $pdo = new PDO(
            $this->config->getDsn(),
            $this->config->getDBUser(),
            $this->config->getDBPassword()
        );
        $statement = $pdo->prepare("SELECT * FROM `orders` WHERE id=:id");
        $statement->execute(array(":id" => $orderID));
        return $query->fetchObject("Order");
    }
    
    public function save($order){/*...*/}
    public function update($order){/*...*/}
    public function delete($order){/*...*/}
}
?>
```

¿Cómo abordamos este caso? Hay varias opciones. 

- Una de ellas es modificar todos los métodos de OrderRepository, pero esto no cumple el principio abierto/cerrado.

- Otra sería crear otra clase que herede de OrderRepository y sobreescribir todos los métodos. Pero no parece muy profesional.

- Otra sería que orderRepository utilice una interfaz e inyectarle la clase concreta en tiempo de ejecución (inversión de dependencias).

```php
<?php
class OrderRepository
{
    private $source;

    public function setSource(IOrderSource $source)
    {
        $this->source = $source;
    }

    public function find($orderID)
    {
        return $this->source->find($orderID);
    }
    
    public function save($order){/*...*/}
    public function update($order){/*...*/}
}

interface IOrderSource
{
    public function find($orderID);
    public function save($order);
    public function update($order);
    public function delete($order);
}

class MySQLOrderSource implements IOrderSource
{
    public function find($orderID);
    public function save($order){/*...*/}
    public function update($order){/*...*/}
    public function delete($order){/*...*/}
}

class ApiOrderSource implements IOrderSource
{
    public function find($orderID);
    public function save($order){/*...*/}
    public function update($order){/*...*/}
    public function delete($order){/*...*/}
}
?>
```

Así podemos cambiar el comportamiento de OrderRepository sin modificar la clase. Si queremos que trabaje contra una base de datos, le pasaremos la clase MySQLOrderSource y si queremos que trabaje contra una API le pasaremos la clase ApiOrderSource, ambas con el mismo interfaz.

Esta solución está basada en el patrón de **inversión de dependencias**.

El patrón de inversión de dependencias también hubiera funcionado en el ejemplo 1.

### Ejemplo 3

Pongamos que hemos programado un Bundle que gestiona la seguridad de nuestra aplicación. Entre otras cosas tiene un controlador que registra a un usuario a través de un servicio también incluido en el Bundle.

```yml
register:
  path:     /register
  defaults: { _controller: LleegoSecurityBundle\Controller\SecurityController:register }
```

```php
class SecurityController extends Controller
{
    public function register()
    {
        // ...
        RegisterService::register($registerData);
        // ...
    }
}
```

El bundle está funcionando a la perfección en varias aplicaciones. Pero en una aplicación concreta queremos que cada vez que se registre un usuario, se anote ese suceso en una tabla LogUserAccess. 

¿Cómo lo resolveríais?

Una solución es copiar el controlador del bundle y hacer que se ejecute un controlador de nuestra aplicación para la ruta /register. En ese controlador añadiríamos nuestro código nuevo:

```yml
register:
  path:     /register
  defaults: { _controller: App\Controller\SecurityController:register }
```

```php
class MySecurityController extends Controller
{
    public function register()
    {
        // ...
        RegisterService::register($registerData);
        $LogUserAccessService->insert($registerData);
        // ...
    }
}
```

Es una chapuza, pero funciona.

No tenemos otra alternativa porque las clases de nuestro bundle no siguen el principio Open/Closed.

Vamos a realizar un pequeño cambio en nuestro Bundle:

```php
class SecurityController extends Controller
{
    public function register($registerService)
    {
        // ...
        $registerService->register($registerData);
        // ...
    }
}
```

Aquí se ha aplicado otra vez la **inversión de dependencias** (es el último de los principios SOLID). Esta modificación nos permitiría cambiar el objeto $registerService del bundle por otro hecho por nosotros que sea compatible: Es decir, crear una clase nueva con un método register($registerData) y decirle al service container de Symfony que inyecte el nuestro en vez del del Bundle.

```php
class SecurityController extends Controller
{
    public function register($registerService)
    {
        // ...
        $registerService->register($registerData); // Este registerService sería el nuestro (MyRegisterService) en vez del del bundle
        // ...
    }
}

class MyRegisterService extends LleegoSecurityBundle\Services\RegisterService {
    public function register($registerData) {
        // ...
        // Código original del servicio del bundle
        // ...

        // ...
        // Código nuevo para insertar registro en la tabla fechasRegistrosUsuarios
        // ...
    }
}
```

La **inversión de dependencias** es una buena táctica para cumplir el principio open/close, pero en este caso concreto nos hace incumplir el principio de responsabilidad única en la clase MiSecurityService.

Además, esta aplicación (aplicación1) quiere utilizar el bundle y además registrar cosas en una **tabla** (servicio1). Pero otra aplicación (aplicación2) quizás quiera escribir en un **fichero de Log** (servicio2). Y otra aplicación (aplicación3) igual lo que quiere es **enviar un correo** a los comerciales avisando de que se ha registrado un nuevo cliente (servicio3). Y otra aplicación (aplicación4) quizás quiere **registrar en fichero de log y también enviar un correo** (servicio4)...

Cada aplicación tendrá que hacerse su propio MiSecurityService, y además, no se pueden aprovechar los servicios entre las aplicaciones porque no hay 2 iguales. A pesar de que la aplicación 4 quiere hacer lo que hacen los servicios 2 y 3, tiene que crearse el suyo propio para que haga las tareas de ambos servicios.

Hay otra solución mejor para este caso. Programar el bundle con **EVENTOS**.

```php
class SecurityController extends Controller
{
    /**
     * @Route("/register")
     */
    public function register($registerService, $eventDispatcher)
    {
        // ...
        $registerService->register($registerData);
        $eventDispatcher->dispatch('userResgistered',$registerData);
        // ...
    }
}
```

De esta forma, cada aplicación puede tener uno o más suscribers que hagan cosas cuando un usuario ha sido registrado. Un suscriber que escriba en la bbdd, otro suscriber que escriba en un fichero, otro suscriber que mande correos...

## ¿Cómo detectar que estamos violando el principio Open/Closed?

Una de las formas más sencillas para detectarlo es darnos cuenta de qué clases modificamos más a menudo. Si cada vez que hay un nuevo requisito o una modificación de los existentes, las mismas clases se ven afectadas, podemos empezar a entender que estamos violando este principio.

## Ejercicios

### Ejercicio 1

Marcador de progreso de descarga de un fichero

```php
class File {
    public $length;
    public $sent;
}

class Progress {
 
    private $file;
 
    function __construct(File $file) {
        $this->file = $file;
    }
 
    function getAsPercent() {
        return $this->file->sent * 100 / $this->file->length;
    }
 
}
```

Uso:

```php
function testItCanGetTheProgressOfAFileAsAPercent() {
    $file = new File();
    $file->setLength(200);
    $file->setSent(0);
 
    $progress = new Progress($file);
    $this->assertEquals(0, $progress->getAsPercent());

    $file->setSent(100);
    $this->assertEquals(50, $progress->getAsPercent());

    $file->setSent(150);
    $this->assertEquals(75, $progress->getAsPercent());

    $file->setSent(2000);
    $this->assertEquals(100, $progress->getAsPercent());
}
```

Se quiere aprovechar la clase Progress para medir también el progreso en porcentaje de objetos Audio y objetos Video:

```php
function testItCanGetTheProgressOfAnAudioAsAPercent() {
    $audio = new Audio();
    $audio->setDuration(200);
    $audio->setPlayed(0);
 
    $progress = new Progress($audio);
    $this->assertEquals(0, $progress->getAsPercent());

    $audio->setPlayed(100);
    $this->assertEquals(50, $progress->getAsPercent());

    $audio->setPlayed(150);
    $this->assertEquals(75, $progress->getAsPercent());

    $audio->setPlayed(200);
    $this->assertEquals(100, $progress->getAsPercent());
}
```

Programar el código de la clase Audio y los cambios necesarios en File y en Progress para cumplir los requisitos. Se pueden añadir otras clases/interfaces.


```php
interface Measurable {
    function getTotal();
    function getCurrent();
}
```

```php
class File implements Measurable {
    public $length;
    public $sent;

    public function getTotal() { 
        return $this->length;
    };

    public function getCurrent() { 
        return $this->sent
    };
}
```

```php
class Audio implements Measurable {
    public $duration;
    public $played;

    public function getTotal() { 
        return $this->duration;
    };

    public function getCurrent() { 
        return $this->played
    };
}
```

```php
class Progress {
 
    private $item;
 
    function __construct(Measurable $item) {
        $this->item = $item;
    }
 
    function getAsPercent() {
        return $this->item->getCurrent() * 100 / $this->item->getTotal();
    }
 
}
```


```php
function testItCanGetTheProgressOfAFileAsAPercent() {
    $file = new File();
    $file->setLength(200);
    $file->setSent(0);
 
    $progress = new Progress($file);
    $this->assertEquals(0, $progress->getAsPercent());

    $file->setSent(100);
    $this->assertEquals(50, $progress->getAsPercent());

    $file->setSent(150);
    $this->assertEquals(75, $progress->getAsPercent());

    $file->setSent(2000);
    $this->assertEquals(100, $progress->getAsPercent());
}

function testItCanGetTheProgressOfAnAudioAsAPercent() {
    $audio = new Audio();
    $audio->setDuration(200);
    $audio->setPlayed(0);
 
    $progress = new Progress($audio);
    $this->assertEquals(0, $progress->getAsPercent());

    $audio->setPlayed(100);
    $this->assertEquals(50, $progress->getAsPercent());

    $audio->setPlayed(150);
    $this->assertEquals(75, $progress->getAsPercent());

    $audio->setPlayed(200);
    $this->assertEquals(100, $progress->getAsPercent());
}
```

## ¿Cuándo debemos cumplir con este principio?

Hay que decir que añadir esta complejidad no siempre compensa, y **como el resto de principios, sólo será aplicable si realmente es necesario**. Si tienes una parte de tu código que es propensa a cambios, plantéate hacerla de forma que un nuevo cambio impacte lo menos posible en el código existente. Normalmente esto no es fácil de saber a priori, por lo que puedes preocuparte por ello cuando tengas que modificarlo, y hacer los cambios necesarios para cumplir este principio en ese momento.

Intentar hacer un código 100% Open/Closed es prácticamente imposible, y puede hacer que sea ilegible e incluso más difícil de mantener. Los principios SOLID son ideas muy potentes, pero hay que aplicarlas donde corresponda y **sin obsesionarnos con cumplirlas en cada punto del desarrollo**. Casi siempre es más sencillo limitarse a usarlas cuando nos haya surgido la necesidad real.