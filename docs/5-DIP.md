# Principio de inversión de dependencias

No dependas de los detalles, depende de abstracciones.

La mejor manera de conseguir esto es utilizando interfaces en lugar de utilizar clases. Está muy ligado con los conceptos de inyección de dependencias o inyección de servicios.

## Ejemplo

Consideremos el siguente pago de un pedido por parte de un cliente:

```php
<?php
class Customer
{
    private $currentOrder = null;

    public function addItem($item){
        if(is_null($this->currentOrder)){
            $this->currentOrder = new Order();
        }
        return $this->currentOrder->addItem($item);
    }
    
    public function deleteItem($item){
        if(is_null($this->currentOrder)){
            return false;
        }
        return $this->currentOrder ->deleteItem($item);
    }

    public function buyItems()
    {    
        if(is_null($this->currentOrder)){
            return false;
        }
        $processor = new OrderProcessor();
        return $processor->checkout($this->currentOrder);    
    }
}

class OrderProcessor
{
    public function checkout($order){/*...*/}
}
```

Todo parece muy lógico y muy sensato. Pero esto no cumple el principio DIP: La clase Customer depende de la clase OrderProcesor.

Si hilamos más fino, tampoco cumple el principio Open/Closed. 

Y si hilamos mucho más fino, un cambio en la forma de construir el objeto OrderProcessor implicaría modificar nuestra clase Customer. Con lo que la clase Customer se ve afectada por una "razón para cambiar" adicional a las previstas, violando el principio de Responsabilidad Única.

Para quitar la dependencia entre Customer y OrderProcesor, recurrimos a las interfaces. Customer dependerá de una Interfaz (una abstracción) en vez de depender de una clase concreta.

Esta dependencia del interfaz se puede implementar a través de métodos setters, a través de un argumento del método buyItems, a través del constructor de Customer o a través de algún contenedor de inyección de dependencias como el Service Container de Symfony.

```php
<?php
class Customer
{
    private $currentOrder = null;

    public function addItem($item){
        if(is_null($this->currentOrder)){
            $this->currentOrder = new Order();
        }
        return $this->currentOrder->addItem($item);
    }
    public function deleteItem($item){
        if(is_null($this->currentOrder)){
            return false;
        }
        return $this->currentOrder ->deleteItem($item);
    }

    public function buyItems(IOrderProcessor $processor)
    {    
        if(is_null($this->currentOrder)){
            return false;
        }
        
        return $processor->checkout($this->currentOrder);    
    }
}

interface IOrderProcessor
{
    public function checkout($order);
}

class OrderProcessor implements IOrderProcessor
{
    public function checkout($order){/*...*/}
}
?>
```

Ahora la clase Customer depende solamente de la abstracción IOrderProcessor y no de la implementación específica. Es decir, ahora los detalles ya no son importantes para Customer.

## Ejercicios

### Ejercicio 1


Solucionar el ejemplo anterior pasando la abstracción IOrderProcessor a través del constructor de Customer.

### Ejercicio 2

Retomemos el ejemplo visto en la sección de SRP

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

Las dos soluciones propuestas entonces fueron estas

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

y 

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

Metámonos dentro de RateIdeaUseCase() y veamos una posible implementación

```php
class RateIdeaUseCase {

  public function __construct() {

  }

  public function execute($ideaId, $rating) {
    try {
      $ideaRepository = new MySQLIdeaRepository(); 
      $idea = $this->ideaRepository->find($ideaId);
    } catch(Exception $e) {
      throw new RepositoryNotAvailableException();
    }

    if (!$idea) {
      throw new IdeaDoesNotExistException();
    }

    try {
      $idea->addRating($rating);
      $ideaRepository->update($idea);
    } catch(Exception $e) {
      throw new RepositoryNotAvailableException();
    }

    return $idea;
  }
```

La clase RateIdeaUseCase estaría acoplada a la clase MySQLIdeaRepository violando el principio de inversión de dependencias.

Hay que deshacer el desacople. RateIdeaUseCase no debe depender de una clase concreta, sino de una abstracción.

Creamos un interfaz:

```php
<?php

interface IdeaRepository
{
  /**
   * @param int $id
   * @return null|Idea
   */
   public function find($id);

   /**
   * @param Idea $idea
   */
   public function update(Idea $idea);
}
```

Hacemos que nuestra clase Repository implemente dicho interfaz

```php
<?php

class MySQLIdeaRepository implements IdeaRepository
{
    ...
}
```

Hacemos que RateIdeaUseCase dependa del interfaz (que es una abstracción, un contrato) en lugar de depender del repositorio concreto:

```php
<?php

class RateIdeaUseCase {

  private $ideaRepository;

  public function __construct(IdeaRepository $ideaRepository) {
    $this->ideaRepository = $ideaRepository;
  }

  public function execute($ideaId, $rating) {
    try {
      $idea = $this->ideaRepository->find($ideaId);
    } catch(Exception $e) {
      throw new RepositoryNotAvailableException();
    }

    if (!$idea) {
      throw new IdeaDoesNotExistException();
    }

    try {
      $idea->addRating($rating);
      $this->ideaRepository->update($idea);
    } catch(Exception $e) {
      throw new RepositoryNotAvailableException();
    }

    return $idea;
  }
}
```

Quien vaya a utilizar RateIdeaUseCase, tendrá que proporcionarle la clase concreta que desee que utilice. 

```php
<?php

class IdeaController extends Controller
{
  public function rateAction() {
    $ideaId = $this->request->getParam('id');
    $rating = $this->request->getParam('rating');

    $ideaRepository = new MySQLIdeaRepository(); 
    $useCase = new RateIdeaUseCase($ideaRepository);
    $response = $useCase->execute($ideaId, $rating);

    $this->redirect('/idea/'.$ideaId);
  }
}
```



## Resumen

Recordemos los consejos de Robert C. Martin:

- Los módulos de alto nivel no deberían depender de módulos de bajo nivel. Ambos deberían depender de abstracciones.
- Las abstracciones no deberían depender de los detalles. Los detalles deberían depender de las abstracciones.

El objetivo del Dependency Inversion Principle (DIP) consiste en reducir las dependencias entre los módulos del código, es decir, alcanzar un **bajo acoplamiento** de las clases.
