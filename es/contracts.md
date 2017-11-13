# Contratos

- [Introducción](#introduction) 
    - [Contratos vs *Facades*](#contracts-vs-facades)
- [Cuándo usar contratos](#when-to-use-contracts) 
    - [Acoplamiento débil](#loose-coupling)
    - [Simplicidad](#simplicity)
- [Cómo utilizar contratos](#how-to-use-contracts)
- [Referencia de contratos](#contract-reference)

<a name="introduction"></a>

## Introducción

Los contratos de Laravel son un conjunto de interfaces que definen los servicios principales del framework. Por ejemplo, el contrato `Illuminate\Contracts\Queue\Queue` define los métodos necesarios para la encolar trabajos, mientras que `Illuminate\Contracts\Mail\Mailer` define los métodos para enviar e-mails.

Cada contrato tiene una implementación correspondiente en el framework. Por ejemplo, Laravel provee una implementación de colas con una variedad de drivers, y una implementación de envío de correos que se apoya en [SwiftMailer](https://swiftmailer.symfony.com/).

Todos los contratos de Laravel se encuentran en [su propio repositorio de GitHub](https://github.com/illuminate/contracts). Esto proporciona un punto de referencia sobre todos los contratos disponibles, así como un paquete desacoplado que puede ser utilizado por desarrolladores de paquetes.

<a name="contracts-vs-facades"></a>

### Contratos vs *Facades*

Las [facades](/docs/{{version}}/facades) de Laravel y los *helper* son una forma sencilla de utilizar los servicios de Laravel sin tener que hacer *type-hint* y resolver contratos fuera del *service container*. En la mayoría de los casos, cada *facade* tiene un contrato equivalente.

A diferencia de las *facades*, que no exigen hacer un *require* en el constructor de la clase, los contratos permiten definir dependencias explícitas para las clases. Algunos desarrolladores prefieren definir explícitamente sus dependencias y por lo tanto prefieren el uso de los contratos, mientras otros desarrolladores prefieren la conveniencia de las *facades*.

> {tip} La mayoría de las aplicaciones estarán bien tanto si se prefieren *facades* como contratos. Sin embargo, si se está desarrollando un paquete, se debería considerar con fuerza el uso de contratos ya que serán más fáciles de "testear" en un contexto de paquete.

<a name="when-to-use-contracts"></a>

## Cuándo usar contratos

Como se discutió anteriormente, gran parte de la decisión de usar contratos o *facades* se reduce al gusto personal y a las preferencias del equipo de desarrollo. Tanto los contratos como las *facades* pueden usarse para crear aplicaciones de Laravel robustas y bien probadas. Mientras se mantengan en foco las responsabilidades de las clase, se notarán muy pocas diferencias entre el uso de *facades* y contratos.

Sin embargo, podrían existir muchas preguntas en relación a los contratos. Por ejemplo, ¿por qué usar interfaces? ¿No lo hacen todo más complicado? Simplificaremos las razones de uso de interfaces a los siguientes títulos: acoplamiento débil y simplicidad.

<a name="loose-coupling"></a>

### Acoplamiento débil

Primero, hay que revisar un código que está fuertemente acoplado a una implementación de caché. Tengamos en cuenta lo siguiente:

    <?php
    
    namespace App\Orders;
    
    class Repository
    {
        /**
         * The cache instance.
         */
        protected $cache;
    
        /**
         * Create a new repository instance.
         *
         * @param  \SomePackage\Cache\Memcached  $cache
         * @return void
         */
        public function __construct(\SomePackage\Cache\Memcached $cache)
        {
            $this->cache = $cache;
        }
    
        /**
         * Retrieve an Order by ID.
         *
         * @param  int  $id
         * @return Order
         */
        public function find($id)
        {
            if ($this->cache->has($id))    {
                //
            }
        }
    }
    

En esta clase, el código está fuertemente acoplado a la implementación de caché. Lo está por que depende directamente de una clase *Cache* concreta de un paquete. Si el API del paquete cambia, el código deberá cambiar también.

Además, si se quiere reemplazar la tecnología de caché subyacente (Memcached) con otra tecnología (Redis), habrá de nuevo que modificar el repositorio. El repositorio no debería conocer mucho sobre quien provee los datos o sobre cómo se proveen.

**En lugar de este enfoque, se puede mejorar nuestro código dependiendo de una simple e independiente interfaz:**

    <?php
    
    namespace App\Orders;
    
    use Illuminate\Contracts\Cache\Repository as Cache;
    
    class Repository
    {
        /**
         * The cache instance.
         */
        protected $cache;
    
        /**
         * Create a new repository instance.
         *
         * @param  Cache  $cache
         * @return void
         */
        public function __construct(Cache $cache)
        {
            $this->cache = $cache;
        }
    }
    

Ahora el código no está acoplado a ningún proveedor concreto, o incluso Laravel. Puesto que el paquete de contratos no contiene implementaciones ni dependencias, fácilmente se pueden escribir implementaciones alternativas a cualquier contrato, permitiendo reemplazar la implementación de caché sin modificar ningún código que ya se esté consumiendo.

<a name="simplicity"></a>

### Simplicidad

Cuando todos los servicios de Laravel están claramente definidos por interfaces, es muy fácil determinar la funcionalidad que ofrece un servicio. **Los contratos sirven como documentación de las características del framework.**

Además, cuando se depende de interfaces simples, el código suele ser más fácil de comprender y mantener. En lugar de rastrear los métodos disponibles en una clase grande y complicada, nos podemos referir a una simple y limpia interfaz.

<a name="how-to-use-contracts"></a>

## Cómo utilizar contratos

Así que, ¿cómo se obtiene una implementación de un contrato? En realidad es bastante simple.

Muchos tipos de clases en Laravel se resuelven a través del [service container](/docs/{{version}}/container), incluyendo controladores, capturadores de eventos, *middleware*, colas de trabajo e incluso *Closures* de ruta. Por lo que, para obtener la implementación de un contrato, simplemente con hacer "type-hint" de la interfaz en el constructor de la clase a resolver.

Por ejemplo, veamos este capturador de eventos:

    <?php
    
    namespace App\Listeners;
    
    use App\User;
    use App\Events\OrderWasPlaced;
    use Illuminate\Contracts\Redis\Database;
    
    class CacheOrderInformation
    {
        /**
         * The Redis database implementation.
         */
        protected $redis;
    
        /**
         * Create a new event handler instance.
         *
         * @param  Database  $redis
         * @return void
         */
        public function __construct(Database $redis)
        {
            $this->redis = $redis;
        }
    
        /**
         * Handle the event.
         *
         * @param  OrderWasPlaced  $event
         * @return void
         */
        public function handle(OrderWasPlaced $event)
        {
            //
        }
    }
    

Cuando se resuelve el capturador, el *service container* leerá la firma del constructor de la clase e inyectará los valores apropiados. Para saber más acerca de registrar cosas en el *service container*, revise [su documentación](/docs/{{version}}/container).

<a name="contract-reference"></a>

## Referencia de contratos

Esta tabla provee una referencia rápida a todos los contratos de Laravel y sus *facades* equivalentes:

| Contrato                                                                                                                                                  | *Facade* referencia       |
| --------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------- |
| [Illuminate\Contracts\Auth\Access\Authorizable](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Access/Authorizable.php)                |  &nbsp;                   |
| [Illuminate\Contracts\Auth\Access\Gate](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Access/Gate.php)                                | `Gate`                    |
| [Illuminate\Contracts\Auth\Authenticatable](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Authenticatable.php)                         |  &nbsp;                   |
| [Illuminate\Contracts\Auth\CanResetPassword](https://github.com/illuminate/contracts/blob/{{version}}/Auth/CanResetPassword.php)                       | &nbsp;                    |
| [Illuminate\Contracts\Auth\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Factory.php)                                         | `Auth`                    |
| [Illuminate\Contracts\Auth\Guard](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Guard.php)                                             | `Auth::guard()`           |
| [Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/{{version}}/Auth/PasswordBroker.php)                           | `Password::broker()`      |
| [Illuminate\Contracts\Auth\PasswordBrokerFactory](https://github.com/illuminate/contracts/blob/{{version}}/Auth/PasswordBrokerFactory.php)             | `Password`                |
| [Illuminate\Contracts\Auth\StatefulGuard](https://github.com/illuminate/contracts/blob/{{version}}/Auth/StatefulGuard.php)                             | &nbsp;                    |
| [Illuminate\Contracts\Auth\SupportsBasicAuth](https://github.com/illuminate/contracts/blob/{{version}}/Auth/SupportsBasicAuth.php)                     | &nbsp;                    |
| [Illuminate\Contracts\Auth\UserProvider](https://github.com/illuminate/contracts/blob/{{version}}/Auth/UserProvider.php)                               | &nbsp;                    |
| [Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Bus/Dispatcher.php)                                     | `Bus`                     |
| [Illuminate\Contracts\Bus\QueueingDispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Bus/QueueingDispatcher.php)                     | `Bus::dispatchToQueue()`  |
| [Illuminate\Contracts\Broadcasting\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/Factory.php)                         | `Broadcast`               |
| [Illuminate\Contracts\Broadcasting\Broadcaster](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/Broadcaster.php)                 | `Broadcast::connection()` |
| [Illuminate\Contracts\Broadcasting\ShouldBroadcast](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/ShouldBroadcast.php)         | &nbsp;                    |
| [Illuminate\Contracts\Broadcasting\ShouldBroadcastNow](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/ShouldBroadcastNow.php)   | &nbsp;                    |
| [Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Factory.php)                                       | `Cache`                   |
| [Illuminate\Contracts\Cache\Lock](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Lock.php)                                             | &nbsp;                    |
| [Illuminate\Contracts\Cache\LockProvider](https://github.com/illuminate/contracts/blob/{{version}}/Cache/LockProvider.php)                             | &nbsp;                    |
| [Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Repository.php)                                 | `Cache::driver()`         |
| [Illuminate\Contracts\Cache\Store](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Store.php)                                           | &nbsp;                    |
| [Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/{{version}}/Config/Repository.php)                               | `Config`                  |
| [Illuminate\Contracts\Console\Application](https://github.com/illuminate/contracts/blob/{{version}}/Console/Application.php)                           | &nbsp;                    |
| [Illuminate\Contracts\Console\Kernel](https://github.com/illuminate/contracts/blob/{{version}}/Console/Kernel.php)                                     | `Artisan`                 |
| [Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/{{version}}/Container/Container.php)                           | `App`                     |
| [Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Cookie/Factory.php)                                     | `Cookie`                  |
| [Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/{{version}}/Cookie/QueueingFactory.php)                     | `Cookie::queue()`         |
| [Illuminate\Contracts\Database\ModelIdentifier](https://github.com/illuminate/contracts/blob/{{version}}/Database/ModelIdentifier.php)                 | &nbsp;                    |
| [Illuminate\Contracts\Debug\ExceptionHandler](https://github.com/illuminate/contracts/blob/{{version}}/Debug/ExceptionHandler.php)                     | &nbsp;                    |
| [Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/{{version}}/Encryption/Encrypter.php)                         | `Crypt`                   |
| [Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Events/Dispatcher.php)                               | `Event`                   |
| [Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Cloud.php)                                 | `Storage::cloud()`        |
| [Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Factory.php)                             | `Storage`                 |
| [Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Filesystem.php)                       | `Storage::disk()`         |
| [Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/{{version}}/Foundation/Application.php)                     | `App`                     |
| [Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/{{version}}/Hashing/Hasher.php)                                     | `Hash`                    |
| [Illuminate\Contracts\Http\Kernel](https://github.com/illuminate/contracts/blob/{{version}}/Http/Kernel.php)                                           | &nbsp;                    |
| [Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/{{version}}/Logging/Log.php)                                           | `Log`                     |
| [Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/{{version}}/Mail/MailQueue.php)                                     | `Mail::queue()`           |
| [Illuminate\Contracts\Mail\Mailable](https://github.com/illuminate/contracts/blob/{{version}}/Mail/Mailable.php)                                       | &nbsp;                    |
| [Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/{{version}}/Mail/Mailer.php)                                           | `Mail`                    |
| [Illuminate\Contracts\Notifications\Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Notifications/Dispatcher.php)                 | `Notification`            |
| [Illuminate\Contracts\Notifications\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Notifications/Factory.php)                       | `Notification`            |
| [Illuminate\Contracts\Pagination\LengthAwarePaginator](https://github.com/illuminate/contracts/blob/{{version}}/Pagination/LengthAwarePaginator.php)   | &nbsp;                    |
| [Illuminate\Contracts\Pagination\Paginator](https://github.com/illuminate/contracts/blob/{{version}}/Pagination/Paginator.php)                         | &nbsp;                    |
| [Illuminate\Contracts\Pipeline\Hub](https://github.com/illuminate/contracts/blob/{{version}}/Pipeline/Hub.php)                                         | &nbsp;                    |
| [Illuminate\Contracts\Pipeline\Pipeline](https://github.com/illuminate/contracts/blob/{{version}}/Pipeline/Pipeline.php)                               | &nbsp;                    |
| [Illuminate\Contracts\Queue\EntityResolver](https://github.com/illuminate/contracts/blob/{{version}}/Queue/EntityResolver.php)                         | &nbsp;                    |
| [Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Factory.php)                                       | `Queue`                   |
| [Illuminate\Contracts\Queue\Job](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Job.php)                                               | &nbsp;                    |
| [Illuminate\Contracts\Queue\Monitor](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Monitor.php)                                       | `Queue`                   |
| [Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Queue.php)                                           | `Queue::connection()`     |
| [Illuminate\Contracts\Queue\QueueableCollection](https://github.com/illuminate/contracts/blob/{{version}}/Queue/QueueableCollection.php)               | &nbsp;                    |
| [Illuminate\Contracts\Queue\QueueableEntity](https://github.com/illuminate/contracts/blob/{{version}}/Queue/QueueableEntity.php)                       | &nbsp;                    |
| [Illuminate\Contracts\Queue\ShouldQueue](https://github.com/illuminate/contracts/blob/{{version}}/Queue/ShouldQueue.php)                               | &nbsp;                    |
| [Illuminate\Contracts\Redis\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Redis/Factory.php)                                       | `Redis`                   |
| [Illuminate\Contracts\Routing\BindingRegistrar](https://github.com/illuminate/contracts/blob/{{version}}/Routing/BindingRegistrar.php)                 | `Route`                   |
| [Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/{{version}}/Routing/Registrar.php)                               | `Route`                   |
| [Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/{{version}}/Routing/ResponseFactory.php)                   | `Response`                |
| [Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/{{version}}/Routing/UrlGenerator.php)                         | `URL`                     |
| [Illuminate\Contracts\Routing\UrlRoutable](https://github.com/illuminate/contracts/blob/{{version}}/Routing/UrlRoutable.php)                           | &nbsp;                    |
| [Illuminate\Contracts\Session\Session](https://github.com/illuminate/contracts/blob/{{version}}/Session/Session.php)                                   | `Session::driver()`       |
| [Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Arrayable.php)                               | &nbsp;                    |
| [Illuminate\Contracts\Support\Htmlable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Htmlable.php)                                 | &nbsp;                    |
| [Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Jsonable.php)                                 | &nbsp;                    |
| [Illuminate\Contracts\Support\MessageBag](https://github.com/illuminate/contracts/blob/{{version}}/Support/MessageBag.php)                             | &nbsp;                    |
| [Illuminate\Contracts\Support\MessageProvider](https://github.com/illuminate/contracts/blob/{{version}}/Support/MessageProvider.php)                   | &nbsp;                    |
| [Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Renderable.php)                             | &nbsp;                    |
| [Illuminate\Contracts\Support\Responsable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Responsable.php)                           | &nbsp;                    |
| [Illuminate\Contracts\Translation\Loader](https://github.com/illuminate/contracts/blob/{{version}}/Translation/Loader.php)                             | &nbsp;                    |
| [Illuminate\Contracts\Translation\Translator](https://github.com/illuminate/contracts/blob/{{version}}/Translation/Translator.php)                     | `Lang`                    |
| [Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Factory.php)                             | `Validator`               |
| [Illuminate\Contracts\Validation\ImplicitRule](https://github.com/illuminate/contracts/blob/{{version}}/Validation/ImplicitRule.php)                   | &nbsp;                    |
| [Illuminate\Contracts\Validation\Rule](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Rule.php)                                   | &nbsp;                    |
| [Illuminate\Contracts\Validation\ValidatesWhenResolved](https://github.com/illuminate/contracts/blob/{{version}}/Validation/ValidatesWhenResolved.php) | &nbsp;                    |
| [Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Validator.php)                         | `Validator::make()`       |
| [Illuminate\Contracts\View\Engine](https://github.com/illuminate/contracts/blob/{{version}}/View/Engine.php)                                           | &nbsp;                    |
| [Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/{{version}}/View/Factory.php)                                         | `View`                    |
| [Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/{{version}}/View/View.php)                                               | `View::make()`            |