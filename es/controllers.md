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

## Controladores básicos

<a name="defining-controllers"></a>

### Definir controladores

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

## *Middleware* y controladores

A las rutas de los controladores se les puede asignar [middleware](/docs/{{version}}/middleware) del siguiente modo:

    Route::get('profile', 'UserController@show')->middleware('auth');
    

Sin embargo, es más conveniente especificar el *middleware* en el constructor del controlador. Utilizando el método `middleware` desde el constructor del controlador, se puede asignar un *middleware* a las acciones del controlador. Incluso se puede restringir el *middleware* a únicamente ciertos métodos:

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
    

Los controladores también permiten registrar *middleware* usando un *Closure* o función anónima. Esto proporciona una forma conveniente de definir un *middleware* para un solo controlador sin definir una clase *middleware* completa:

    $this->middleware(function ($request, $next) {
        // ...
    
        return $next($request);
    });
    

> {tip} Se puede asignar un *middleware* a un subconjunto de acciones del controlador; sin embargo, esto puede indicar que el controlador está creciendo demasiado. En su lugar, se recomienda dividir el controlador en controladores más pequeños.

<a name="resource-controllers"></a>

## Controladores de recursos – *resource controllers*

El *routing* de recursos de Laravel asigna las rutas "CRUD" típicas a un controlador con una sola línea de código. Por ejemplo, la creación de un controlador que gestiona todas las peticiones HTTP sobre "photos" (fotos) almacenadas por nuestra aplicación. Utilizando el comando de Artisan `make:controller`, se puede crear un controlador rápidamente:

    php artisan make:controller PhotoController --resource
    

El comando generará un controlador en el archivo `app/Http/Controllers/PhotoController.php`. El controlador incluirá un método para cada una de las operaciones disponibles para el recurso.

A continuación, se puede registrar una ruta de recursos para el controlador:

    Route::resource('photos', 'PhotoController');
    

Esta única declaración crea varias rutas que gestionan los diferentes métodos sobre un recurso. El controlador generado incluirá los métodos ya declarados para cada una de estas acciones, incluyendo notas que informan sobre que URIs y verbos gestionan.

Se pueden registrar varios controladores de recursos a la vez pasando una *array* al método `resources`:

    Route::resources([
        'photos' => 'PhotoController',
        'posts' => 'PostController'
    ]);
    

#### Acciones gestionadas por controladores de recursos

| Verbo     | URI                    | Acción  | Nombre de Ruta |
| --------- | ---------------------- | ------- | -------------- |
| GET       | `/photos`              | index   | photos.index   |
| GET       | `/photos/create`       | create  | photos.create  |
| POST      | `/photos`              | store   | photos.store   |
| GET       | `/photos/{photo}`      | show    | photos.show    |
| GET       | `/photos/{photo}/edit` | edit    | photos.edit    |
| PUT/PATCH | `/photos/{photo}`      | update  | photos.update  |
| DELETE    | `/photos/{photo}`      | destroy | photos.destroy |

#### Especificar el modelo del recurso

Si se está utilizando el *route model binding* y se desea que los métodos del controlador de recursos incluyan un *type-hint* de una instancia del modelo, se puede usar la opción `--model` al generar el controlador:

    php artisan make:controller PhotoController --resource --model=Photo
    

#### Suplantación de métodos en formularios

Los formularios HTML no pueden realizar peticiones `PUT`, `PATCH`, o `DELETE`, para hacerlo se necesita agregar un campo oculto `_method` para suplantar estos verbos HTTP. El *helper* `method_field` permite crear este campo de forma rápida:

    {{ method_field('PUT') }}
    

<a name="restful-partial-resource-routes"></a>

### Rutas de recursos parciales

Cuando se declara una ruta de recursos, se puede especificar un subconjunto de acciones que el controlador debe manejar en lugar del conjunto completo de acciones predeterminadas:

    Route::resource('photo', 'PhotoController', ['only' => [
        'index', 'show'
    ]]);
    
    Route::resource('photo', 'PhotoController', ['except' => [
        'create', 'store', 'update', 'destroy'
    ]]);
    

#### Rutas de recursos para API

Al declarar rutas de recursos que consumirá un API, normalmente se querrá excluir rutas que presenten plantillas HTML, como `create` y `edit`. Para su comodidad, se puede usar el método `apiResource` para excluir automáticamente estas dos rutas:

    Route::apiResource('photo', 'PhotoController');
    

Se pueden registrar varios controladores de recursos de API a la vez pasando una *array* al método `apiResources`:

    Route::apiResources([
        'photos' => 'PhotoController',
        'posts' => 'PostController'
    ]);
    

<a name="restful-naming-resource-routes"></a>

### Nombrar rutas de recursos

Por defecto, todas las acciones de los controladores de recursos tienen un nombre de ruta; sin embargo, se puede sobrescribir este nombre pasando un *array* `names` con sus opciones:

    Route::resource('photo', 'PhotoController', ['names' => [
        'create' => 'photo.build'
    ]]);
    

<a name="restful-naming-resource-route-parameters"></a>

### Nombrar parámetros en rutas de recursos

Por defecto, `Route::resource` crea los parámetros de ruta para las rutas de recursos utilizando la versión "singular" del nombre del recurso. Se puede sobrescribir esto fácilmente por recurso pasando `parameters` en el *array* de opciones. El *array* de `parameters` debe ser un *array* asociativo de los nombres de los recursos y el nombre de su parámetro:

    Route::resource('user', 'AdminUserController', ['parameters' => [
        'user' => 'admin_user'
    ]]);
    

El ejemplo anterior genera las siguientes URIs para la ruta `show` del recurso:

    /user/{admin_user}
    

<a name="restful-localizing-resource-uris"></a>

### Traducir las URIs de los recursos

Por defecto, `Route::resource` crea las URIs de los recursos usando verbos en inglés. Si se necesita traducir los verbos de las acciones `create` y `edit`, se puede usar el método `Route::resourceVerbs`. Se puede hacer esto en el método `boot` de nuestro `AppServiceProvider`:

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
    

Una vez que se han personalizado los verbos, una ruta de recursos como `Route::resource('fotos', 'PhotoController')` producirá las siguientes URIs:

    /fotos/crear
    
    /fotos/{foto}/editar
    

<a name="restful-supplementing-resource-controllers"></a>

### Complementar a los controladores de recursos

Si es necesario agregar rutas adicionales a un controlador de recursos más allá de las predeterminadas, se deben definir antes de llamar a `Route::resource`; de otro modo, las rutas definidas por el método `resource` pueden prevalecer sobre las rutas suplementarias:

    Route::get('photos/popular', 'PhotoController@method');
    
    Route::resource('photos', 'PhotoController');
    

> {tip} Recuerde mantener sus controladores enfocados. Si se encuentra rutinariamente necesitando métodos fuera del conjunto típico de las acciones de recursos, considere dividir su controlador en dos controladores más pequeños.

<a name="dependency-injection-and-controllers"></a>

## Inyección de dependencias & controladores

#### Inyección en constructores

El [service container](/docs/{{version}}/container) de Laravel se utiliza para resolver todos los controladores. Como resultado, se puede *type-hint* (firma del método) cualquier dependencia que el controlador pueda tener en su constructor. Las dependencias declaradas se resuelven automáticamente y se inyectan en la instancia del controlador:

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
    

Y por supuesto, se puede incluir cualquier [Contracto de Laravel](/docs/{{version}}/contracts). Si el contenedor puede resolverlo, se puede utilizar en la firma del constructor. Dependiendo de la aplicación, inyectar las dependencias dentro del controlador puede ofrecer mejor control sobre el *testing*.

#### Inyección en métodos

Además de inyectar en constructores, se puede hacer *type-hint* de dependencias en los métodos del controlador. Un caso de uso muy común para la inyección es la de la instancia de `Illuminate\Http\Request` dentro de los métodos del controlador:

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
    

Si el método del controlador también espera datos de entrada de un parámetro en la ruta se deben listar los parámetros de ruta después de las otras dependencias. Por ejemplo, si la ruta está definida así:

    Route::put('user/{id}', 'UserController@update');
    

Se podría hacer *type-hint* de `Illuminate\Http\Request` y acceder al parámetro de ruta `id` definiendo el método del controlador de la siguiente forma:

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

## Caché de rutas

> {note} Las rutas basadas en *Closures* o funciones anónimas no se pueden almacenar en caché. Para utilizar el almacenamiento en caché de rutas, debe convertir las rutas de *Closure* en controladores.

Si la aplicación usa exclusivamente rutas basadas en controladores, se puede aprovechar la caché de rutas de Laravel. Utilizando la caché de rutas se reducirá drásticamente el tiempo que toma la aplicación en registrar todas las rutas. En algunos casos, ¡el registro de rutas puede ser hasta 100x más rápido. Para generar una caché de rutas, simplemente hay que ejecutar el comando de Artisan `route:cache`:

    php artisan route:cache
    

Tras ejecutar el comando, el archivo de rutas en caché se cargará en cada solicitud. Recordar, si se añaden nuevas rutas se debe generar de nuevo una nueva caché. Es por esto, se recomienda utilizar el comando `route:cache` únicamente en el entorno de producción.

Se puede utilizar el comando `route:clear` para limpiar las rutas en caché:

    php artisan route:clear