# Middleware

- [Introduccion](#introduction)
- [Definir Middleware](#defining-middleware)
- [Registrar Middleware](#registering-middleware) 
    - [Middleware Global](#global-middleware)
    - [Asignar Middleware a Rutas](#assigning-middleware-to-routes)
    - [Grupos de Middleware](#middleware-groups)
- [Middleware con Parámetros](#middleware-parameters)
- [Middleware Terminable](#terminable-middleware)

<a name="introduction"></a>

## Introduccion

Los middleware proporcionan una herramienta para filtrar las peticiones HTTP que entran a la aplicación. Por ejemplo, Laravel incluye un middleware que verifica si el actual usuario de la aplicación esta autenticado. Si el usuario no esta autenticado, el middleware redirigirá al usuario a la vista de login. Sin embargo, si el usuario esta autenticado, el middleware permitirá que la petición continue y se ejecute en la aplicación.

Por supuesto, se pueden crear middleware adicionales para realizar otro tipo tareas además de la autenticación. Un CORS middleware podría usarse para añadir los headers adecuados a todas las respuestas de la aplicación. Un logging middleware podría registrar en el log todas las peticiones hechas a la aplicación.

Hay una gran cantidad de middleware incluidos en Laravel framework, incluyendo middleware para autenticación y protección CSRF. Todos estos middleware están ubicados en el directorio `app/Http/Middleware` .

<a name="defining-middleware"></a>

## Definir Middleware

La forma más sencilla de crear un nuevo middleware es utilizar el comando Artisan `make:middleware` :

    php artisan make:middleware CheckAge
    

Este comando creará un nuevo middleware llamado `CheckAge` en el directorio `app/Http/Middleware`. Este middleware solo permitirá acceso a la ruta si la `age` suministrada es mayor a 200. De otra forma, el middleware redirigirá a los usuarios de vuelta a la URI `home`.

    <?php
    
    namespace App\Http\Middleware;
    
    use Closure;
    
    class CheckAge
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            if ($request->age <= 200) {
                return redirect('home');
            }
    
            return $next($request);
        }
    }
    

Como puede verse, si la `age` es menor o igual a `200`, el middleware retornará un redirect HTTP al cliente; de otra manera la request será ejecutada por la aplicación. Para pasar la petición hacia abajo en la aplicación (permitir al middleware "pasar"), simplemente hay que llamar al callback `$next` con `$request`.

Lo mejor es imaginar los middleware como una serie de "capas" por las que las requests HTTP deben pasar antes de que lleguen a la aplicación. Cada capa puede examinar la request e incluso rechazarla por completo.

### Middleware Antes & Después

Que el middleware se ejecute antes o después de que la petición entre en la aplicación depende del uso del middleware en si mismo. Por ejemplo, el siguiente middleware ejecuta algunas tareas antes de que la petición sea gestionada por la aplicación:

    <?php
    
    namespace App\Http\Middleware;
    
    use Closure;
    
    class BeforeMiddleware
    {
        public function handle($request, Closure $next)
        {
            // Perform action
    
            return $next($request);
        }
    }
    

Sin embargo, este middleware ejecuta las tareas después de que la petición haya sido gestionada por la aplicación:

    <?php
    
    namespace App\Http\Middleware;
    
    use Closure;
    
    class AfterMiddleware
    {
        public function handle($request, Closure $next)
        {
            $response = $next($request);
    
            // Perform action
    
            return $response;
        }
    }
    

<a name="registering-middleware"></a>

## Registrar Middleware

<a name="global-middleware"></a>

### Middleware Global

Si se desea que un middleware se ejecute en todas las peticiones HTTP de la aplicación, simplemente debe listar el middleware en la propiedad `$middleware` de la clase `app/Http/Kernel.php`.

<a name="assigning-middleware-to-routes"></a>

### Asignar Middleware a Rutas

Si se desea que el middleware se ejecute en rutas especificas, primero debe asignarse al middleware un identificador en el archivo `app/Http/Kernel.php`. La propiedad `$routeMiddleware` de esta clase contiene registros de middleware incluidos por Laravel por defecto. Para agregar middleware personalizados, simplemente debe añadirse a la lista el nuevo middleware y asignarle el identificador de acceso rápido que se desee. Por ejemplo:

    // Within App\Http\Kernel Class...
    
    protected $routeMiddleware = [
        'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    ];
    

Una vez que el middleware ha sido registrado en el kernel HTTP, se puede utilizar el identificador de `middleware` para asignarlo a una ruta:

    Route::get('admin/profile', function () {
        //
    })->middleware('auth');
    

Se puede además asignar varios middleware a una ruta:

    Route::get('/', function () {
        //
    })->middleware('first', 'second');
    

Al asignar un middleware, se puede también pasar el nombre completo de la clase:

    use App\Http\Middleware\CheckAge;
    
    Route::get('admin/profile', function () {
        //
    })->middleware(CheckAge::class);
    

<a name="middleware-groups"></a>

### Grupos de Middleware

En ocasiones es útil agrupar varios middleware sobre un mismo identificador haciendo la asignación a rutas mucho más simple. Esto se puede hacer utilizando la propiedad `$middlewareGroups` del kernel HTTP.

Por defecto, Laravel incluye los grupos de middleware `web` y `api` que contienen el middleware común que se suele aplicar a las rutas de web UI y API:

    /**
     * The application's route middleware groups.
     *
     * @var array
     */
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],
    
        'api' => [
            'throttle:60,1',
            'auth:api',
        ],
    ];
    

Los grupos de middleware se pueden asignar a rutas y acciones de controladores utilizando la misma sintaxis que un middleware individual. De nuevo, los grupos únicamente permiten añadir varios middleware de una vez:

    Route::get('/', function () {
        //
    })->middleware('web');
    
    Route::group(['middleware' => ['web']], function () {
        //
    });
    

> {tip} El grupo `web` se aplica directamente al archivo `routes/web.php` a través del `RouteServiceProvider`.

<a name="middleware-parameters"></a>

## Middleware con Parámetros

Los middleware pueden recibir parámetros adicionales. Por ejemplo, si la aplicación necesita verificar que el usuario autenticado tiene asignado cierto "rol" antes de ejecutar una acción, puede crearse un middleware `CheckRole` que reciba el nombre del rol como parámetro adicional.

Los parámetros adicionales del middleware deben ser pasados después de argumento `$next` :

    <?php
    
    namespace App\Http\Middleware;
    
    use Closure;
    
    class CheckRole
    {
        /**
         * Handle the incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @param  string  $role
         * @return mixed
         */
        public function handle($request, Closure $next, $role)
        {
            if (! $request->user()->hasRole($role)) {
                // Redirect...
            }
    
            return $next($request);
        }
    
    }
    

Los parámetros del middleware pueden ser especificados se define la ruta, separando el nombre del middleware y los parámetros con `:`. Multiples parámetros deben ser separados por comas:

    Route::put('post/{id}', function ($id) {
        //
    })->middleware('role:editor');
    

<a name="terminable-middleware"></a>

## Middleware Terminable

En ocasiones, un middleware necesita realizar algunas acciones después de que la respuesta HTTP ha sido enviada al navegador. Por ejemplo, el middleware "session" incluido por defecto en Laravel, registra los datos de sesión *después* de que la respuesta haya sido enviada al navegador. Si se define un método `terminate` en el middleware, se ejecutará automáticamente después de que la respuesta se haya enviado al navegador.

    <?php
    
    namespace Illuminate\Session\Middleware;
    
    use Closure;
    
    class StartSession
    {
        public function handle($request, Closure $next)
        {
            return $next($request);
        }
    
        public function terminate($request, $response)
        {
            // Store the session data...
        }
    }
    

El método `terminate` debe recibir tanto la petición como la respuesta. Una vez que se ha definido un middleware terminable, este debe añadirse a la lista de rutas o middleware globales en `app/Http/Kernel.php`.

Cuando se llama al método `terminate` en el middleware, Laravel resolverá una nueva instancia del middleware desde el [service container](/docs/{{version}}/container). Si se desea usar la misma instancia cuando los métodos `handle` y `terminate` son llamados, debe registrarse en el container el middleware usando el método `singleton` del container.