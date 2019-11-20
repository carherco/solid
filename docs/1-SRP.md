# Principio de Responsabilidad Única

**Una clase debería tener una, y solo una, razón para cambiar**

El principio de Responsabilidad Única nos viene a decir que un objeto debe realizar una única cosa. Es muy habitual, si no prestamos atención a esto, que acabemos teniendo clases que tienen varias responsabilidades lógicas a la vez.

Un ejemplo:

```php
class Employee
{
    public getName()
    public setName()
    public save()
    public calculatePay()
    public reportHours()
}
```

## ¿Cómo detectar si estamos violando el Principio de Responsabilidad Única?

La respuesta a esta pregunta es bastante subjetiva. **Sin necesidad de obsesionarnos** con ello, podemos detectar situaciones en las que una clase podría dividirse en varias:

### En una misma clase están involucradas dos capas de la arquitectura

Esta puede ser difícil de ver sin experiencia previa. En toda arquitectura, por simple que sea, debería haber una capa de presentación, una de lógica de negocio y otra de persistencia. Si mezclamos responsabilidades de dos capas en una misma clase, será un buen indicio.

- Objetos que se renderizan ellos mismos
- Objetos que se guardan ellos mismos


### El número de métodos públicos

Si una clase hace muchas cosas, lo más probable es que tenga muchos métodos públicos, y que tengan poco que ver entre ellos. Detecta cómo puedes agruparlos para separarlos en distintas clases. Algunos de los puntos siguientes te pueden ayudar.

### Los métodos que usan cada uno de los campos de esa clase

Si tenemos dos campos, y uno de ellos se usa en unos cuantos métodos y otro en otros cuantos, esto puede estar indicando que cada campo con sus correspondientes métodos podrían formar una clase independiente. Normalmente esto estará más difuso y habrá métodos en común, porque seguramente esas dos nuevas clases tendrán que interactuar entre ellas.

### Por el número de "uses"

Si necesitamos importar demasiadas clases externas para hacer nuestro trabajo, es posible que estemos haciendo trabajo de más. También ayuda fijarse a qué paquetes pertenecen esos "use". Si vemos que NO se agrupan con facilidad, puede que nos esté avisando de que estamos haciendo cosas muy diferentes.

### Nos cuesta testear la clase

Si no somos capaces de escribir tests unitarios sobre ella, o no conseguimos el grado de granularidad que nos gustaría, es momento de plantearse dividir la clase en dos.

### Cada vez que escribes una nueva funcionalidad, esa clase se ve afectada

Si una clase se modifica a menudo, es porque está involucrada en demasiadas cosas.

### Por el número de líneas

A veces es tan sencillo como eso. Si una clase es demasiado grande, intenta dividirla en clases más manejables.


En general no hay reglas de oro para estar 100% seguros. La práctica te irá haciendo ver cuándo es recomendable que cierto código se mueva a otra clase, pero estos indicios te ayudarán a detectar algunos casos donde tengas dudas.

## Ejercicios

## Ejercicio 1: 

```php
<?php
class Order
{
    public function calculateTotalSum(){/*...*/}
    public function getItems(){/*...*/}
    public function getItemCount(){/*...*/}
    public function addItem($item){/*...*/}
    public function deleteItem($item){/*...*/}

    public function printOrder(){/*...*/}
    public function showOrder(){/*...*/}

    public function load(){/*...*/}
    public function save(){/*...*/}
    public function update(){/*...*/}
    public function delete(){/*...*/}
}
?>
```

Solución

```php
<?php
class Order
{
    public function calculateTotalSum(){/*...*/}
    public function getItems(){/*...*/}
    public function getItemCount(){/*...*/}
    public function addItem($item){/*...*/}
    public function deleteItem($item){/*...*/}
}

class OrderRepository
{
    public function load($orderID){/*...*/}
    public function save($order){/*...*/}
    public function update($order){/*...*/}
    public function delete($order){/*...*/}
}

class OrderViewer
{
    public function printOrder($order){/*...*/}
    public function showOrder($order){/*...*/}
}
?>
```

### Ejercicio 2: 

```php
class User {
 
    ...

    protected function formatResponse(User $user) {
        return [
          "name"     => $user->name,
          "userName" => $user->username,
          "rank"     => $user->rank,
          "score"    => $user->score
        ];
    }
 
    protected function validateUser(User $user) {
        if ($user) {
          return true;
        } else {
          throw new UnknownUserException("User doesn`t exist");
        }
    }
 
    protected function fetchUserFromDatabase($userId) {
        return $this->userRepository->find($userId);
    }
 
    ...
}
```

Solución

```php
<?php
class User {
    ...
}

class UserFormatter {
    protected function toArray(User $user) {
        return [
          "name"     => $user->name,
          "userName" => $user->username,
          "rank"     => $user->rank,
          "score"    => $user->score
        ];
    }
}

class UserValidator {
    protected function validate(User $user) {
        if ($user->name) {
          return true;
        } else {
          throw new NameRequiredException("Name is required");
        }
    }
}

class UserRepository {
    protected function fetchUserFromDatabase($userId) {
        return $this->userRepository->find($userId);
    } 
}
```

### Ejercicio 3

```php
class Book {
 
    protected function getTitle() {
        return "A Great Book";
    }
 
    protected function getAuthor() {
        return "John Doe";
    }
 
    protected function turnPage() {
        // pointer to next page
    }

    function getCurrentPage() {
        return "current page content";
    }
 
    protected function printCurrentPage() {
        echo "current page content";
    }
}
```

Solución

```php
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
}

class Printer {
 
    function printPage($page) {
        echo $page;
    }
 
}
```

Con un poco más de esfuerzo, podemos añadir más métodos de impresión

```php
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
}
 
interface Printer {
 
    function printPage($page);
}
 
class PlainTextPrinter implements Printer {
 
    function printPage($page) {
        echo $page;
    }
 
}
 
class HtmlPrinter implements Printer {
 
    function printPage($page) {
        echo '<div style="single-page">' . $page . '</div>';
    }
 
}
```

### Ejercicio 4

```php
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
    function saveToFile() {
        $filename = '/documents/'. $this->getTitle(). ' - ' . $this->getAuthor();
        file_put_contents($filename, serialize($this));
    }
 
}
```

Solución 

```php
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
}
 
class BookFilePersistence {
 
    function save(Book $book) {
        $filename = '/documents/' . $book->getTitle() . ' - ' . $book->getAuthor();
        file_put_contents($filename, serialize($book));
    }
 
}
```

## Ejercicio 5:

```php
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
    function getLocation() {
        // returns the position in the library
        // ie. shelf number & room number
        return $libraryMap->findBookBy($this->getTitle(), $this->getAuthor());
    }
 
}
```

Solución 

```php
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
}
 
class BookLocator {
 
    function locate(Book $book) {
        // returns the position in the library
        // ie. shelf number & room number
        return $libraryMap->findBookBy($book->getTitle(), $book->getAuthor());
    }
 
}
```

## Ejercicio 6

Aunque una clase únicamente tenga un método, no nos asegura que tenga solamente una única razón para cambiar.


```php
class IdeaController extends Controller
{
  public function rateAction() {
    // Getting parameters from the request
    $ideaId = $this->request->getParam('id');
    $rating = $this->request->getParam('rating');
    // Building database connection
    $db = new PDO([
      'host' => 'localhost',
      'username' => 'demo',
      'password' => '',
      'dbname' => 'demo'
    ]);
    // Finding the idea in the database
    $sql = 'SELECT * FROM ideas WHERE idea_id = ?';
    $stmt = $db->prepare($sql);
    $stmt->bindParam(1, $ideaId);
    $stmt->execute();
    $row = $stmt->fetch();
    if (!$row) {
      throw new Exception('Idea does not exist');
    }
    // Building the idea from the database
    $idea = new Idea();
    $idea->setId($row['id']);
    $idea->setTitle($row['title']);
    $idea->setDescription($row['description']);
    $idea->setRating($row['rating']);
    $idea->setVotes($row['votes']);
    $idea->setAuthor($row['email']);
    // Add user rating (internally it will increment votes by 1)
    $idea->addRating($rating);
    // Save it to the database
    $sql = 'UPDATE ideas SET votes = ?, rating = ? WHERE idea_id = ?';
    $stmt = $db->prepare($sql);
    $stmt->bindParam(1, $ideaId);
    $stmt->bindParam(2, $idea->getVotes());
    $stmt->bindParam(3, $idea->getRating());
    $stmt->execute();
    // Redirect to view idea page
    $this->redirect('/idea/'.$ideaId);
  }
}
```

Solución "aceptable"

```php
class IdeaController extends Controller
{
  public function rateAction() {
    // Getting parameters from the request
    $ideaId = $this->request->getParam('id');
    $rating = $this->request->getParam('rating');

    // Retrieving the idea from the database
    $ideaRepository = new MySQLIdeaRepository(); 
    $ideaEntity = $ideaRepository->find($ideaId);

    // Add user rating (internally it will increment votes by 1)
    $ideaEntity->addRating($rating);

    // Save it to the database
    $ideaRepository->save($ideaEntity);
    
    // Redirect to view idea page
    $this->redirect('/idea/'.$ideaEntity->getId());
  }
}
```

Solución "casi perfecta"

```php
class IdeaController extends Controller
{
  public function rateAction() {
    // Getting parameters from the request
    $ideaId = $this->request->getParam('id');
    $rating = $this->request->getParam('rating');

    // Executing work
    $useCase = new RateIdeaUseCase($ideaId, $rating);
    $ideaEntity = $useCase->execute();

    // Redirect to view idea page
    $this->redirect('/idea/'.$ideaEntity->getId());
  }
}
```



## Anti-patrones

- God Object: un objeto que lo sabe todo y lo hace todo.

Ejemplo (caso real):

```php
<?php

class Lector 
{
    public function getNombre() { /* ... */ }
    public function getApellido1() { /* ... */ }
    public function getApellido2() { /* ... */ }
    //... getters ...

    public function setNombre($value) { /* ... */ }
    public function getApellido1($value) { /* ... */ }
    public function getApellido2($value) { /* ... */ }
    //... setters ...

    public function getNombreApellidos()
    {
      return $this->getNombre()." ".$this->getApellido1()." ".$this->getApellido2();
    }

    public function getApellidosNombre()
    {
      return $this->getApellido1()." ".$this->getApellido2().", ".$this->getNombre();
    }
    
    public function getCodigoNombreApellidos()
    {
      return $this->getCodigolector().' - '.$this->getNombre()." ".$this->getApellido1()." ".$this->getApellido2();
    }
    
    public function getUsername()
    {
      return $this->getUsuario()->getUsername();
    }
    
    /**
     * Devuelve el nombre del curso del lector
     * 
     * @return string
     */
    public function getCurso() {

      $curso = $this->getCurso(); 
      if($curso instanceof BibCursos) {
        return $curso->getDescripcion();
      } else {
        return "";
      }

    }
      
    public function countPrestamos($nEstado = null, $criteria = null, $distinct = false, $con = null)
    {
        // ...
    }

    /**
     * Returns the number of related Prestamoss.
     */
    public function countPrestamosActivos($criteria = null, $distinct = false, $con = null)
    {
        // ...
    }

    /**
     * Devuelve el número de prestamos retrasados sin devolver
     */
    public function countPrestamosActivosRetrasados($criteria = null, $distinct = false, $con = null)
    {
        // ...
    }
    
    /**
     * Devuelve los prestamos activos retrasados
     */
    public function getPrestamosActivosRetrasados($criteria = null, $distinct = false, $con = null)
    {
        // ...
    }

    /**
     * Returns the number of related Reservas.
     */
    public function countReservasActivas($criteria = null, $distinct = false, $con = null)
    {
        // ...
    }
    
    /**
     * Devuelve el número de reservas Activas de un tipo de ejemplar concreto
     */
    public function countReservasTipoEjemplarActivas($idTipoEjemplar)
    {
        // ...
    }

    public function countSolicitudesPrestamosPendientes()
    {
        // ...
    }

    public function  __toString()
    {
        // ...
    }

    public function tienePrestamosActivos()
    {
        return ($this->countPrestamosActivos() > 0);
    }

    public function tienePrestamosActivosRetrasados()
    {
        return ($this->countPrestamosActivosRetrasados() > 0);
    }

    public function tienePrestamos()
    {
        // ...
    }

    public function tienePrestadoEjemplar($idEjemplar)
    {
        // ...
    }

    public function tieneReservadoEjemplar($idEjemplar)
    {
        // ...
    }

    public function tieneOpiniones()
    {
        // ...
    }

    public function getIdUsuario()
    {
        // ...
    }

    public static function getNumLecturas()
    {
        // ...
    }

    public function posicionReserva($eIdEjemplar)
    {
        // ...
    }
    
    static function getLimiteReservas($idTipoEjemplar)
    {
        // ...
    }

    /**
     * Devuelve el número de prestamos retrasados sin devolver
     * 
     * Es un alias de countPrestamosActivosRetrasados()
     * 
     * @return int
     */
    public function getNumPrestamosRetrasadosSinDevolver()
    {
        return $this->countPrestamosActivosRetrasados();
    }

    /**
     * Devuelve los días de sanción que le quedan al lector por cumplir.
     */
    public function getDiasSancion()
    {
      $dias = 0;

      if(($this->getFechasancion('Y-m-d') != null) && ($this->getFechasancion('Y-m-d') > date('Y-m-d')))
      {
        $dias = (strtotime($this->getFechasancion('Y-m-d')) - strtotime(date('Y-m-d'))) / 86400; //86400 = 60*60624
      }

      return $dias;
    }
    
    public function estaSancionado() {
        return $this->getDiasSancion()>0;
    }

    public function delete()
    {
        // ...
    }

    public function esBorrable()
    {
        return !$this->tienePrestamosActivos();
    }
    
   public function getEmail()
   {
       // ...
   }

    static function insert($data)
    {
        // ...
    }

}
```