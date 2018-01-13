# Facades

- [Introducción](#introduction)
- [Cuando utilizar *facades* ](#when-to-use-facades) 
    - [*Facades* vs inyección de dependencias](#facades-vs-dependency-injection)
    - [*Facades* vs *helpers*](#facades-vs-helper-functions)
- [Cómo funcionan las *facades* ](#how-facades-work)
- [*Facades* en tiempo real](#real-time-facades)
- [Referencia de *facades* ](#facade-class-reference)

<a name="introduction"></a>

## Introducción

Las *facades* proveen de una interfaz "estática" a las clases disponibles en el [service container](/docs/{{version}}/container). Laravel incluye varias *facades* que proveen acceso a casi todas las características del framework. Las *facades* de Laravel sirven como "proxies estáticos" a las clases subyacentes del *service container*, proporcionando una sintaxis expresiva mientras que se proporciona más *testabilidad* y flexibilidad que con métodos estáticos tradicionales.

Todas las *facades* de Laravel se definen en el *namespace* `Illuminate\Support\Facades`. Se puede acceder a una *facade* así:

    use Illuminate\Support\Facades\Cache;
    
    Route::get('/cache', function () {
        return Cache::get('key');
    });
    

A través de la documentación de Laravel, muchos de los ejemplos utilizarán *facades* para demostrar varias características del framework.

<a name="when-to-use-facades"></a>

## Cuando utilizar *facades* 

Las *facades* tienen muchas ventajas. Proporcionan una sintaxis fácil de recordar que permite utilizar las características de Laravel sin tener que recordar nombres de clases largos para inyectar o configurados manualmente. Además, por su único uso de métodos de PHP dinámicos, son fáciles de *testear*.

Sin embargo, se debe tener cuidado al usar facades. El principal peligro de las *facades* es el deslizamiento del alcance de clase. Como las *facades* son tan fáciles de usar y no requieren inyección, puede ser fácil dejar que las clases sigan creciendo y usar muchas fachadas en una sola clase. Al usar la inyección de dependencia, este potencial se ve mitigado por los comentarios visuales que un gran constructor le indica que su clase está creciendo demasiado. Por lo tanto, al usar facades, se debe prestar especial atención al tamaño de la clase para que su alcance de su responsabilidad se mantenga estrecho.

> {tip} Al crear un paquete de terceros que interactúa con Laravel, es mejor inyectar [Laravel contracts](/docs/{{version}}/contracts) en lugar de utilizar de *facades* . Dado que los paquetes se construyen fuera de Laravel, no se tendrá acceso a los helpers de pruebas de los *facades* de Laravel.

<a name="facades-vs-dependency-injection"></a>

### *Facades* vs inyección de dependencias

Uno de los principales beneficios de la inyección de dependencias es la capacidad de intercambiar implementaciones de la clase inyectada. Esto es útil durante las pruebas ya que puede inyectar un *mock* o un *stub* y verificar que se han llamado varios métodos en el *stub*.

Normalmente, no sería posible hacer un *mock* o *stub* de un método de clase verdaderamente estático. Sin embargo, dado que las *facades* usan métodos dinámicos para realizar llamadas de métodos proxy a objetos resueltos desde el * service container*, en realidad podemos probar *facades* de la misma manera que probaríamos una instancia de clase inyectada. Por ejemplo, dada la siguiente ruta:

    use Illuminate\Support\Facades\Cache;
    
    Route::get('/cache', function () {
        return Cache::get('key');
    });
    

Podemos escribir la siguiente prueba para verificar que el método `Cache::get` ha sido llamado con el argumento esperado:

    use Illuminate\Support\Facades\Cache;
    
    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');
    
        $this->visit('/cache')
             ->see('value');
    }
    

<a name="facades-vs-helper-functions"></a>

### *Facades* vs. Funciones *Helper*

Además de las *facades*, Laravel incluye gran variedad de funciones de ayuda o "helper" que pueden realizar tareas comunes tales como generar vistas, disparar eventos, despachar trabajos o enviar respuestas HTTP. Muchas de esas funciones de ayuda realizan una función equivalente a la de una *facade* correspondiente. Por ejemplo, esta llamada *facade* y esta función helper son equivalentes:

    return View::make('profile');
    
    return view('profile');
    

No hay absolutamente ninguna diferencia práctica entre *facades* y funciones *helper*. Cuando se usan funciones *helper*, puedes hacer pruebas exactamente igual que si usaras la *facade* correspondiente. Por ejemplo, dada la siguiente ruta:

    Route::get('/cache', function () {
        return cache('key');
    });
    

En el fondo, el *helper* `cache` llama al método `get` en la clase subyacente a la *facade* `Cache`. Entonces, aunque estamos usando la función auxiliar, podemos escribir la siguiente prueba para verificar que el método fue llamado con el argumento que esperábamos:

    use Illuminate\Support\Facades\Cache;
    
    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');
    
        $this->visit('/cache')
             ->see('value');
    }
    

<a name="how-facades-work"></a>

## Cómo funcionan las *facades* 

En el contexto de una aplicación Laravel, una *facade* es una clase que provee acceso a un objeto del *container*. La maquinaria que realiza este trabajo es la clase `Facade`. Tanto las *facades* de Laravel como cualquier facade personalizado, heredarán de la clase base `Illuminate\Support\Facades\Facade`.

La clase base `Facade` hace uso del método mágico `__callStatic()` para canalizar las llamadas de la *facade* al objeto resuelto. En el siguiente ejemplo, se realiza una llamada al sistema de caché de Laravel. Al echar un vistazo a este código, se podría suponer que se llama al método estático `get` en la clase `Cache`:

    <?php
    
    namespace App\Http\Controllers;
    
    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Cache;
    
    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            $user = Cache::get('user:'.$id);
    
            return view('profile', ['user' => $user]);
        }
    }
    

Tener en cuenta que al comienzo del archivo se está "importando" la *facade* `Cache`. Esta *facade* sirve como un proxy para acceder a la implementación subyacente de la interfaz `Illuminate\Contracts\Cache\Factory`. Cualquier llamada realizada a través de la *facade* se pasará a la instancia subyacente del servicio de caché de Laravel.

Si se observa la clase `Illuminate\Support\Facades\Cache`, no existe el método `get`:

    class Cache extends Facade
    {
        /**
         * Get the registered name of the component.
         *
         * @return string
         */
        protected static function getFacadeAccessor() { return 'cache'; }
    }
    

En su lugar, la *facade* `Cache` hereda de la clase base `Facade` y define el método `getFacadeAccessor()`. Recordar, la responsabilidad de este método es retornar el nombre de un *binding* en el *service container*. Cuando se hace uso de cualquier método estático de la *facade* `Cache`, Laravel resolverá el *binding* `cache` del [service container](/docs/{{version}}/container) y ejecutará el método solicitado (en este caso, `get`) en el objeto resuelto.

<a name="real-time-facades"></a>

## *Facades* en tiempo real

Usando *facades* en tiempo real, se puede tratar cualquier clase en su aplicación como si fuera una *facade*. Para ilustrar cómo se puede usar esto, examinemos una alternativa. Por ejemplo, supongamos que nuestro modelo `Podcast` tiene un método `publish`. Sin embargo, para publicar el podcast, necesitamos inyectar una instancia de `Publisher`:

    <?php
    
    namespace App;
    
    use App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;
    
    class Podcast extends Model
    {
        /**
         * Publish the podcast.
         *
         * @param  Publisher  $publisher
         * @return void
         */
        public function publish(Publisher $publisher)
        {
            $this->update(['publishing' => now()]);
    
            $publisher->publish($this);
        }
    }
    

Inyectar una implementación de *publisher* en el método nos permite probar fácilmente el método de forma aislada ya que podemos simular el editor inyectado. Sin embargo, nos exige pasar siempre una instancia de editor cada vez que llamemos al método `publish`. Al utilizar *facades* en tiempo real, podemos mantener la misma capacidad de prueba sin tener que pasar explícitamente una instancia de `Publisher`. Para generar una fachada en tiempo real, simplemente se coloca `Facades` como prefijo en el namespace de la clase importada:

    <?php
    
    namespace App;
    
    use Facades\App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;
    
    class Podcast extends Model
    {
        /**
         * Publish the podcast.
         *
         * @return void
         */
        public function publish()
        {
            $this->update(['publishing' => now()]);
    
            Publisher::publish($this);
        }
    }
    

Cuando se usa un *facade* en tiempo real, la implementación de *publisher* se resolverá fuera del *service container* utilizando la parte de la interfaz o nombre de clase que aparece después del prefijo `Facades`. Al testear, podemos utilizar los helpers de prueba de *facades* incorporados de Laravel para simular esta llamada a método:

    <?php
    
    namespace Tests\Feature;
    
    use App\Podcast;
    use Tests\TestCase;
    use Facades\App\Contracts\Publisher;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    
    class PodcastTest extends TestCase
    {
        use RefreshDatabase;
    
        /**
         * A test example.
         *
         * @return void
         */
        public function test_podcast_can_be_published()
        {
            $podcast = factory(Podcast::class)->create();
    
            Publisher::shouldReceive('publish')->once()->with($podcast);
    
            $podcast->publish();
        }
    }
    

<a name="facade-class-reference"></a>

## Referencia de Clases Facade

A continuación se detalla una lista de *facades* y sus clases subyacentes. Resultará útil para acceder de forma rápida a la documentación API para la raíz de una *facade* concreta. La clave [service container binding](/docs/{{version}}/container) se incluye cuando aplica.

| Facade               | Clase                                                                                                                                          | Service Container Binding |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------- |
| App                  | [Illuminate\Foundation\Application](https://laravel.com/api/{{version}}/Illuminate/Foundation/Application.html)                              | `app`                     |
| Artisan              | [Illuminate\Contracts\Console\Kernel](https://laravel.com/api/{{version}}/Illuminate/Contracts/Console/Kernel.html)                         | `artisan`                 |
| Auth                 | [Illuminate\Auth\AuthManager](https://laravel.com/api/{{version}}/Illuminate/Auth/AuthManager.html)                                          | `auth`                    |
| Auth (Instance)      | [Illuminate\Contracts\Auth\Guard](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Guard.html)                                 | `auth.driver`             |
| Blade                | [Illuminate\View\Compilers\BladeCompiler](https://laravel.com/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html)                 | `blade.compiler`          |
| Broadcast            | [Illuminate\Contracts\Broadcasting\Factory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Broadcasting/Factory.html)             | &nbsp;                    |
| Broadcast (Instance) | [Illuminate\Contracts\Broadcasting\Broadcaster](https://laravel.com/api/{{version}}/Illuminate/Contracts/Broadcasting/Broadcaster.html)     | &nbsp;                    |
| Bus                  | [Illuminate\Contracts\Bus\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html)                         | &nbsp;                    |
| Cache                | [Illuminate\Cache\CacheManager](https://laravel.com/api/{{version}}/Illuminate/Cache/CacheManager.html)                                      | `cache`                   |
| Cache (Instance)     | [Illuminate\Cache\Repository](https://laravel.com/api/{{version}}/Illuminate/Cache/Repository.html)                                          | `cache.store`             |
| Config               | [Illuminate\Config\Repository](https://laravel.com/api/{{version}}/Illuminate/Config/Repository.html)                                        | `config`                  |
| Cookie               | [Illuminate\Cookie\CookieJar](https://laravel.com/api/{{version}}/Illuminate/Cookie/CookieJar.html)                                          | `cookie`                  |
| Crypt                | [Illuminate\Encryption\Encrypter](https://laravel.com/api/{{version}}/Illuminate/Encryption/Encrypter.html)                                  | `encrypter`               |
| DB                   | [Illuminate\Database\DatabaseManager](https://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html)                          | `db`                      |
| DB (Instance)        | [Illuminate\Database\Connection](https://laravel.com/api/{{version}}/Illuminate/Database/Connection.html)                                    | `db.connection`           |
| Event                | [Illuminate\Events\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html)                                        | `events`                  |
| File                 | [Illuminate\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html)                                | `files`                   |
| Gate                 | [Illuminate\Contracts\Auth\Access\Gate](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Access/Gate.html)                    | &nbsp;                    |
| Hash                 | [Illuminate\Contracts\Hashing\Hasher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html)                         | `hash`                    |
| Lang                 | [Illuminate\Translation\Translator](https://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html)                              | `translator`              |
| Log                  | [Illuminate\Log\Writer](https://laravel.com/api/{{version}}/Illuminate/Log/Writer.html)                                                      | `log`                     |
| Mail                 | [Illuminate\Mail\Mailer](https://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html)                                                    | `mailer`                  |
| Notification         | [Illuminate\Notifications\ChannelManager](https://laravel.com/api/{{version}}/Illuminate/Notifications/ChannelManager.html)                  | &nbsp;                    |
| Password             | [Illuminate\Auth\Passwords\PasswordBrokerManager](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBrokerManager.html) | `auth.password`           |
| Password (Instance)  | [Illuminate\Auth\Passwords\PasswordBroker](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html)               | `auth.password.broker`    |
| Queue                | [Illuminate\Queue\QueueManager](https://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html)                                      | `queue`                   |
| Queue (Instance)     | [Illuminate\Contracts\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Contracts/Queue/Queue.html)                               | `queue.connection`        |
| Queue (Base Class)   | [Illuminate\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html)                                                    | &nbsp;                    |
| Redirect             | [Illuminate\Routing\Redirector](https://laravel.com/api/{{version}}/Illuminate/Routing/Redirector.html)                                      | `redirect`                |
| Redis                | [Illuminate\Redis\RedisManager](https://laravel.com/api/{{version}}/Illuminate/Redis/RedisManager.html)                                      | `redis`                   |
| Redis (Instance)     | [Illuminate\Redis\Connections\Connection](https://laravel.com/api/{{version}}/Illuminate/Redis/Connections/Connection.html)                 | `redis.connection`        |
| Request              | [Illuminate\Http\Request](https://laravel.com/api/{{version}}/Illuminate/Http/Request.html)                                                  | `request`                 |
| Response             | [Illuminate\Contracts\Routing\ResponseFactory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html)       | &nbsp;                    |
| Response (Instance)  | [Illuminate\Http\Response](https://laravel.com/api/{{version}}/Illuminate/Http/Response.html)                                                | &nbsp;                    |
| Route                | [Illuminate\Routing\Router](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html)                                              | `router`                  |
| Schema               | [Illuminate\Database\Schema\Builder](https://laravel.com/api/{{version}}/Illuminate/Database/Schema/Builder.html)                           | &nbsp;                    |
| Session              | [Illuminate\Session\SessionManager](https://laravel.com/api/{{version}}/Illuminate/Session/SessionManager.html)                              | `session`                 |
| Session (Instance)   | [Illuminate\Session\Store](https://laravel.com/api/{{version}}/Illuminate/Session/Store.html)                                                | `session.store`           |
| Storage              | [Illuminate\Filesystem\FilesystemManager](https://laravel.com/api/{{version}}/Illuminate/Filesystem/FilesystemManager.html)                  | `filesystem`              |
| Storage (Instance)   | [Illuminate\Contracts\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Contracts/Filesystem/Filesystem.html)           | `filesystem.disk`         |
| URL                  | [Illuminate\Routing\UrlGenerator](https://laravel.com/api/{{version}}/Illuminate/Routing/UrlGenerator.html)                                  | `url`                     |
| Validator            | [Illuminate\Validation\Factory](https://laravel.com/api/{{version}}/Illuminate/Validation/Factory.html)                                      | `validator`               |
| Validator (Instance) | [Illuminate\Validation\Validator](https://laravel.com/api/{{version}}/Illuminate/Validation/Validator.html)                                  | &nbsp;                    |
| View                 | [Illuminate\View\Factory](https://laravel.com/api/{{version}}/Illuminate/View/Factory.html)                                                  | `view`                    |
| View (Instance)      | [Illuminate\View\View](https://laravel.com/api/{{version}}/Illuminate/View/View.html)                                                        | &nbsp;                    |