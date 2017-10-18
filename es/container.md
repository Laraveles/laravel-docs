# Service Container

- [Introduccion](#introduction)
- [*Binding* (Enlazado)](#binding) 
    - [Lo Básico de un *Binding*](#binding-basics)
    - [Enlazar Interfaces a Implementaciones](#binding-interfaces-to-implementations)
    - [*Binding* Contextual](#contextual-binding)
    - [Etiquetado](#tagging)
- [Resolución](#resolving) 
    - [El Método *make*](#the-make-method)
    - [Inyección Automática](#automatic-injection)
- [Eventos del Container](#container-events)
- [PSR-11](#psr-11)

<a name="introduction"></a>

## Introducción

El service container de Laravel es una potente herramienta para gestionar las dependencias de clases y llevar a cabo la inyección de estas. La inyección de dependencias es una frase moderna que básicamente significa: las dependencias de la clase se "inyectan" en la clase a través del constructor o, en algunos casos, métodos "setter".

Un simple ejemplo:

    <?php
    
    namespace App\Http\Controllers;
    
    use App\User;
    use App\Repositories\UserRepository;
    use App\Http\Controllers\Controller;
    
    class UserController extends Controller
    {
        /**
         * The user repository implementation.
         *
         * @var UserRepository
         */
        protected $users;
    
        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }
    
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            $user = $this->users->find($id);
    
            return view('user.profile', ['user' => $user]);
        }
    }
    

En este ejemplo, `UserController` necesita obtener usuarios desde una fuente de datos. Por lo tanto, se **inyecta** un servicio que es capaz de ello. En este contexto, el `UserRepository` probablemente utilizará [Eloquent](/docs/{{version}}/eloquent) para obtener la información del usuario de la base de datos. Sin embargo, puesto que el repositorio se inyecta, es muy sencillo intercambiarlo por otra implementación. Se podría además crear un *mock*, o una implementación ficticia de `UserRepository` para las pruebas de la aplicación.

Una comprensión profunda del service container de Laravel es básica para crear una aplicación grande y potente, así como para contribuir al core de Laravel.

<a name="binding"></a>

## Binding (Enlazado)

<a name="binding-basics"></a>

### Lo Básico de un *Binding*

Casi todos los *bindings* del *service container* se registrarán en [service providers](/docs/{{version}}/providers), por lo que la mayoría de estos ejemplos mostrarán el uso del *container* en ese contexto.

> {tip} No hay necesidad de enlazar clases en el *container* si no dependen de una interfaz. El *container* no necesita conocer cómo crear esos objetos, puesto que puede resolverlos automáticamente utilizando el *reflection* nativo de PHP.

#### *Bindings* Simples

En un *service provider* siempre se tendrá acceso al *container* a través de la propiedad `$this->app`. Se puede registrar un *binding* utilizando el método `bind`, pasando el nombre de la clase o interfaz que se desee registrar acompañado de un `Closure` que retornará una instancia de la clase:

    $this->app->bind('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });
    

Tener en cuenta que se recibe el *container* en sí mismo como argumento. Se puede utilizar para resolver sub-dependencias del objeto que se está creando.

#### Enlazar un Singleton

El método `singleton` enlaza una clase o interfaz al *container* y que únicamente debe ser resuelta una vez. Una vez que se ha resuelto un *binding singleton*, se devolverá la misma instancia del objeto a las subsecuentes llamadas en el *container*:

    $this->app->singleton('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });
    

#### Enlazar Instancias

Se puede enlazar la instancia de un objeto existente al container utilizando el método `instance`. La instancia proporcionada se retornará en las futuras llamadas:

    $api = new HelpSpot\API(new HttpClient);
    
    $this->app->instance('HelpSpot\API', $api);
    

#### *Bindings* Primitivos

Se puede dar el caso de una clase que recibe otras clases inyectadas pero que además necesita de algún valor primitivo como un entero. Se puede utilizar un enlazado contextual para inyectar el valor que la clase necesite:

    $this->app->when('App\Http\Controllers\UserController')
              ->needs('$variableName')
              ->give($value);
    

<a name="binding-interfaces-to-implementations"></a>

### Enlazar Interfaces a Implementaciones

Una característica muy potente del service container es la capacidad de enlazar una interfaz a una implementación concreta. Por ejemplo, una interfaz `EventPusher` y una implementación `ReidsEventPusher`. Una vez que se ha programado la implementación `RedisEventPusher` de la interfaz, se puede registrar en el service container así:

    $this->app->bind(
        'App\Contracts\EventPusher',
        'App\Services\RedisEventPusher'
    );
    

Esto indica al *container* que debe inyectar la clase `RedisEventPusher` cuando una clase necesite una implementación de `EventPusher`. Ahora se puede *type-hint* la interfaz `EventPusher` en un constructor, o cualquier otra localización donde las dependencias se inyecten por el service container:

    use App\Contracts\EventPusher;
    
    /**
     * Create a new class instance.
     *
     * @param  EventPusher  $pusher
     * @return void
     */
    public function __construct(EventPusher $pusher)
    {
        $this->pusher = $pusher;
    }
    

<a name="contextual-binding"></a>

### *Binding* Contextual

A veces se pueden tener dos clases que utilicen la misma interfaz, pero inyectar diferentes implementaciones en cada clase. Por ejemplo, dos controladores pueden depender de diferentes implementaciones del [contrato](/docs/{{version}}/contracts) `Illuminate\Contracts\Filesystem\Filesystem`. Laravel provee de una simple y fluida interfaz para definir este comportamiento:

    use Illuminate\Support\Facades\Storage;
    use App\Http\Controllers\PhotoController;
    use App\Http\Controllers\VideoController;
    use Illuminate\Contracts\Filesystem\Filesystem;
    
    $this->app->when(PhotoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('local');
              });
    
    $this->app->when(VideoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('s3');
              });
    

<a name="tagging"></a>

### Etiquetado

En ocasiones, puede ser necesario resolver toda una "categoría" concreta de bindings. Por ejemplo, quizá se esta desarrollando un agregador de informes que recibe un array de diferentes implementaciones de la interfaz `Report`. Tras registrar las implementaciones `Report`, se les puede asignar una etiqueta utilizando el método `tag`:

    $this->app->bind('SpeedReport', function () {
        //
    });
    
    $this->app->bind('MemoryReport', function () {
        //
    });
    
    $this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');
    

Una vez que los servicios se han etiquetado, se pueden resolver utilizando el método `tagget`:

    $this->app->bind('ReportAggregator', function ($app) {
        return new ReportAggregator($app->tagged('reports'));
    });
    

<a name="resolving"></a>

## Resolución

<a name="the-make-method"></a>

#### El Método `make`

Se puede utilizar el método `make` para resolver la instancia de una clase desde el *container*. Este método aceptará el nombre de la clase o interfaz a resolver:

    $api = $this->app->make('HelpSpot\API');
    

Se puede utilizar la función global `resolve` para acceder a la variable `$app` si no se tiene acceso a ella en alguna parte del código:

    $api = resolve('HelpSpot\API');
    

Si alguna/s dependencias de la clase no se pueden resolver a través del *container*, se podrían inyectar pasándolas como un *array* asociativo al método `makeWith`:

    $api = $this->app->makeWith('HelpSpot\API', ['id' => 1]);
    

<a name="automatic-injection"></a>

#### Inyección Automática

Alternativamente y muy importante, se puede "*type-hint*" una dependencia en el constructor de cualquier clase que se resuelve en el *container*, incluyendo [controladores](/docs/{{version}}/controllers), [event listeners](/docs/{{version}}/events), [queue jobs](/docs/{{version}}/queues), [middleware](/docs/{{version}}/middleware), y más. En la práctica, así es como la mayoría de objetos se resuelven por el *container*.

Por ejemplo, se puede *type-hint* un repositorio de la aplicación en el constructor de un controlador. El repositorio se resolverá automáticamente y se inyectará en la clase:

    <?php
    
    namespace App\Http\Controllers;
    
    use App\Users\Repository as UserRepository;
    
    class UserController extends Controller
    {
        /**
         * The user repository instance.
         */
        protected $users;
    
        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }
    
        /**
         * Show the user with the given ID.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            //
        }
    }
    

<a name="container-events"></a>

## Eventos del Container

El *service container* dispara un evento cada vez que resuelve un objeto. Se puede capturar utilizando el método `resolving`:

    $this->app->resolving(function ($object, $app) {
        // Called when container resolves object of any type...
    });
    
    $this->app->resolving(HelpSpot\API::class, function ($api, $app) {
        // Called when container resolves objects of type "HelpSpot\API"...
    });
    

Como se puede apreciar, el objeto a ser resuelto se pasará al *callback*, permitiendo establecer cualquier propiedad adicional al objeto antes de que se entregue.

<a name="psr-11"></a>

## PSR-11

El *service container* de Laravel implementa la interfaz [PSR-11](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md). Por lo tanto, se puede *type-hint* (sugerencia de tipo) la interfaz PSR-11 del *container* para obtener una instancia del *container* de Laravel:

    use Psr\Container\ContainerInterface;
    
    Route::get('/', function (ContainerInterface $container) {
        $service = $container->get('Service');
    
        //
    });
    

> {note} El método `get` lanzará una excepción si el identificador no se ha enlazado en el *container* explícitamente.