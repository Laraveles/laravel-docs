# Controladores

- [Introducción](#introduction)
- [Controladores básicos](#basic-controllers) 
    - [Definir controladores](#defining-controllers)
    - [Controladores & *namespaces*](#controllers-and-namespaces)
    - [Controladores de acción única](#single-action-controllers)
- [*Middleware* y controladores](#controller-middleware)
- [Controladores de recursos – *resource controllers*](#resource-controllers) 
    - [Rutas de recursos parciales](#restful-partial-resource-routes)
    - [Nombrar rutas de recursos](#restful-naming-resource-routes)
    - [Nombrar parámetros en rutas de recursos](#restful-naming-resource-route-parameters)
    - [Traducir las URIs de los recursos](#restful-localizing-resource-uris)
    - [Complementar a los controladores de recursos](#restful-supplementing-resource-controllers)
- [Inyección de dependencias & controladores](#dependency-injection-and-controllers)
- [Caché de rutas](#route-caching)

<a name="introduction"></a>

## Introducción

En lugar de definir toda la lógica para la gestión de una petición dentro de *Closures* o funciones anónimas en los archivos de rutas, se puede organizar este comportamiento en unas clases llamadas Controladores (*controllers*). Los controladores pueden agrupar la lógica de gestión de peticiones relacionadas en una única clase. Estos controladores se encuentran normalmente en el directorio `app/Http/Controllers`.

<a name="basic-controllers"></a>

## Basic Controllers

<a name="defining-controllers"></a>

### Defining Controllers

A continuación se muestra un ejemplo de una clase de controlador básico. Tenga en cuenta que el controlador hereda de la clase de controlador base incluida con Laravel. La clase base provee de una serie de métodos útiles como el método `middleware`, que se puede usar para adjuntar un *middleware* a las acciones del controlador:

    <?php
    
    namespace App\Http\Controllers;
    
    use App\User;
    use App\Http\Controllers\Controller;
    
    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }
    

Se puede apuntar una ruta a la acción de este controlador así:

    Route::get('user/{id}', 'UserController@show');
    

Ahora, cuando una petición concuerda con la URI de la ruta, se ejecutará el método `show` de la clase `UserController`. Por supuesto, los parámetros de la ruta se pasarán también a este método.

> {tip} Los controladores no **requieren** heredar la clase base. Sin embargo, no se tendrá acceso a las características como los métodos `middleware`, `validate`, y `dispatch`.

<a name="controllers-and-namespaces"></a>

### Controladores & *namespaces*

Es muy importante tener en cuenta que no es necesario especificar el *namespace* completo del controlador cuando se define la ruta del controlador. Como el `RouteServiceProvider` carga los archivos de ruta dentro de un grupo de rutas que contiene el *namespace*, únicamente especificamos la porción del nombre de clase que viene después del *namespace* `App\Http\Controllers`.

Si se prefiere anidar u organizar los controladores más profundos que el directorio `App\Http\Controllers`, simplemente se debe utilizar el nombre de la clase relativo a `App\Http\Controllers` como *namespace* raíz. Por lo que, si el la clase del controlador es `App\Http\Controllers\Photos\AdminController`, se debe registrar la siguiente ruta:

    Route::get('foo', 'Photos\AdminController@method');
    

<a name="single-action-controllers"></a>

### Controladores de acción única

Si se necesita, se puede definir un controlador que únicamente gestione una única acción, simplemente es necesario colocar el método `__invoke` dentro del controlador:

    <?php
    
    namespace App\Http\Controllers;
    
    use App\User;
    use App\Http\Controllers\Controller;
    
    class ShowProfile extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function __invoke($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }
    

Cuando se registra una ruta de un controlador de acción única, no se necesita especificar ningún método en la ruta:

    Route::get('user/{id}', 'ShowProfile');
    

<a name="controller-middleware"></a>

## Controller Middleware

[Middleware](/docs/{{version}}/middleware) may be assigned to the controller's routes in your route files:

    Route::get('profile', 'UserController@show')->middleware('auth');
    

However, it is more convenient to specify middleware within your controller's constructor. Using the `middleware` method from your controller's constructor, you may easily assign middleware to the controller's action. You may even restrict the middleware to only certain methods on the controller class:

    class UserController extends Controller
    {
        /**
         * Instantiate a new controller instance.
         *
         * @return void
         */
        public function __construct()
        {
            $this->middleware('auth');
    
            $this->middleware('log')->only('index');
    
            $this->middleware('subscribed')->except('store');
        }
    }
    

Controllers also allow you to register middleware using a Closure. This provides a convenient way to define a middleware for a single controller without defining an entire middleware class:

    $this->middleware(function ($request, $next) {
        // ...
    
        return $next($request);
    });
    

> {tip} You may assign middleware to a subset of controller actions; however, it may indicate your controller is growing too large. Instead, consider breaking your controller into multiple, smaller controllers.

<a name="resource-controllers"></a>

## Resource Controllers

Laravel resource routing assigns the typical "CRUD" routes to a controller with a single line of code. For example, you may wish to create a controller that handles all HTTP requests for "photos" stored by your application. Using the `make:controller` Artisan command, we can quickly create such a controller:

    php artisan make:controller PhotoController --resource
    

This command will generate a controller at `app/Http/Controllers/PhotoController.php`. The controller will contain a method for each of the available resource operations.

Next, you may register a resourceful route to the controller:

    Route::resource('photos', 'PhotoController');
    

This single route declaration creates multiple routes to handle a variety of actions on the resource. The generated controller will already have methods stubbed for each of these actions, including notes informing you of the HTTP verbs and URIs they handle.

You may register many resource controllers at once by passing an array to the `resources` method:

    Route::resources([
        'photos' => 'PhotoController',
        'posts' => 'PostController'
    ]);
    

#### Actions Handled By Resource Controller

| Verb      | URI                    | Action  | Route Name     |
| --------- | ---------------------- | ------- | -------------- |
| GET       | `/photos`              | index   | photos.index   |
| GET       | `/photos/create`       | create  | photos.create  |
| POST      | `/photos`              | store   | photos.store   |
| GET       | `/photos/{photo}`      | show    | photos.show    |
| GET       | `/photos/{photo}/edit` | edit    | photos.edit    |
| PUT/PATCH | `/photos/{photo}`      | update  | photos.update  |
| DELETE    | `/photos/{photo}`      | destroy | photos.destroy |

#### Specifying The Resource Model

If you are using route model binding and would like the resource controller's methods to type-hint a model instance, you may use the `--model` option when generating the controller:

    php artisan make:controller PhotoController --resource --model=Photo
    

#### Spoofing Form Methods

Los formularios HTML no pueden realizar peticiones `PUT`, `PATCH`, o `DELETE`, para hacerlo se necesita agregar un campo oculto `_method` para suplantar estos verbos HTTP. The `method_field` helper can create this field for you:

    {{ method_field('PUT') }}
    

<a name="restful-partial-resource-routes"></a>

### Partial Resource Routes

When declaring a resource route, you may specify a subset of actions the controller should handle instead of the full set of default actions:

    Route::resource('photo', 'PhotoController', ['only' => [
        'index', 'show'
    ]]);
    
    Route::resource('photo', 'PhotoController', ['except' => [
        'create', 'store', 'update', 'destroy'
    ]]);
    

#### API Resource Routes

When declaring resource routes that will be consumed by APIs, you will commonly want to exclude routes that present HTML templates such as `create` and `edit`. For convenience, you may use the `apiResource` method to automatically exclude these two routes:

    Route::apiResource('photo', 'PhotoController');
    

You may register many API resource controllers at once by passing an array to the `apiResources` method:

    Route::apiResources([
        'photos' => 'PhotoController',
        'posts' => 'PostController'
    ]);
    

<a name="restful-naming-resource-routes"></a>

### Naming Resource Routes

By default, all resource controller actions have a route name; however, you can override these names by passing a `names` array with your options:

    Route::resource('photo', 'PhotoController', ['names' => [
        'create' => 'photo.build'
    ]]);
    

<a name="restful-naming-resource-route-parameters"></a>

### Naming Resource Route Parameters

By default, `Route::resource` will create the route parameters for your resource routes based on the "singularized" version of the resource name. You can easily override this on a per resource basis by passing `parameters` in the options array. The `parameters` array should be an associative array of resource names and parameter names:

    Route::resource('user', 'AdminUserController', ['parameters' => [
        'user' => 'admin_user'
    ]]);
    

The example above generates the following URIs for the resource's `show` route:

    /user/{admin_user}
    

<a name="restful-localizing-resource-uris"></a>

### Localizing Resource URIs

By default, `Route::resource` will create resource URIs using English verbs. If you need to localize the `create` and `edit` action verbs, you may use the `Route::resourceVerbs` method. This may be done in the `boot` method of your `AppServiceProvider`:

    use Illuminate\Support\Facades\Route;
    
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Route::resourceVerbs([
            'create' => 'crear',
            'edit' => 'editar',
        ]);
    }
    

Once the verbs have been customized, a resource route registration such as `Route::resource('fotos', 'PhotoController')` will produce the following URIs:

    /fotos/crear
    
    /fotos/{foto}/editar
    

<a name="restful-supplementing-resource-controllers"></a>

### Supplementing Resource Controllers

If you need to add additional routes to a resource controller beyond the default set of resource routes, you should define those routes before your call to `Route::resource`; otherwise, the routes defined by the `resource` method may unintentionally take precedence over your supplemental routes:

    Route::get('photos/popular', 'PhotoController@method');
    
    Route::resource('photos', 'PhotoController');
    

> {tip} Remember to keep your controllers focused. If you find yourself routinely needing methods outside of the typical set of resource actions, consider splitting your controller into two, smaller controllers.

<a name="dependency-injection-and-controllers"></a>

## Dependency Injection & Controllers

#### Constructor Injection

The Laravel [service container](/docs/{{version}}/container) is used to resolve all Laravel controllers. As a result, you are able to type-hint any dependencies your controller may need in its constructor. The declared dependencies will automatically be resolved and injected into the controller instance:

    <?php
    
    namespace App\Http\Controllers;
    
    use App\Repositories\UserRepository;
    
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
    }
    

Of course, you may also type-hint any [Laravel contract](/docs/{{version}}/contracts). If the container can resolve it, you can type-hint it. Depending on your application, injecting your dependencies into your controller may provide better testability.

#### Method Injection

In addition to constructor injection, you may also type-hint dependencies on your controller's methods. A common use-case for method injection is injecting the `Illuminate\Http\Request` instance into your controller methods:

    <?php
    
    namespace App\Http\Controllers;
    
    use Illuminate\Http\Request;
    
    class UserController extends Controller
    {
        /**
         * Store a new user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $name = $request->name;
    
            //
        }
    }
    

If your controller method is also expecting input from a route parameter, simply list your route arguments after your other dependencies. For example, if your route is defined like so:

    Route::put('user/{id}', 'UserController@update');
    

You may still type-hint the `Illuminate\Http\Request` and access your `id` parameter by defining your controller method as follows:

    <?php
    
    namespace App\Http\Controllers;
    
    use Illuminate\Http\Request;
    
    class UserController extends Controller
    {
        /**
         * Update the given user.
         *
         * @param  Request  $request
         * @param  string  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }
    

<a name="route-caching"></a>

## Route Caching

> {note} Closure based routes cannot be cached. To use route caching, you must convert any Closure routes to controller classes.

If your application is exclusively using controller based routes, you should take advantage of Laravel's route cache. Using the route cache will drastically decrease the amount of time it takes to register all of your application's routes. In some cases, your route registration may even be up to 100x faster. To generate a route cache, just execute the `route:cache` Artisan command:

    php artisan route:cache
    

After running this command, your cached routes file will be loaded on every request. Remember, if you add any new routes you will need to generate a fresh route cache. Because of this, you should only run the `route:cache` command during your project's deployment.

You may use the `route:clear` command to clear the route cache:

    php artisan route:clear