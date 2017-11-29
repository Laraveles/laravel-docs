# Rutas

- [Rutas básicas](#basic-routing) 
    - [Redirección de rutas](#redirect-routes)
    - [Rutas con vistas](#view-routes)
- [Rutas con parámetros](#route-parameters) 
    - [Parámetros requeridos](#required-parameters)
    - [Parámetros opcionales](#parameters-optional-parameters)
    - [Expresiones regulares](#parameters-regular-expression-constraints)
- [Nombres de rutas](#named-routes)
- [Grupos de rutas](#route-groups) 
    - [Middleware](#route-group-middleware)
    - [Namespaces](#route-group-namespaces)
    - [Rutas de sub-dominios](#route-group-sub-domain-routing)
    - [Prefijos de rutas](#route-group-prefixes)
- [Enlazar modelos a rutas](#route-model-binding) 
    - [Enlazado implícito](#implicit-binding)
    - [Enlazado explícito](#explicit-binding)
- [Falsear el método del formulario](#form-method-spoofing)
- [Acceder a la ruta actual](#accessing-the-current-route)

<a name="basic-routing"></a>

## Rutas Básicas

La ruta más básica en Laravel únicamente acepta la URI y un `Closure`, ofreciendo una forma muy sencilla y expresiva de definir rutas:

    Route::get('foo', function () {
        return 'Hello World';
    });
    

#### El archivo de rutas por defecto

Todas las rutas de Laravel se definen en los archivos que se encuentran en la carpeta `routes`. El framework carga estos archivos de forma automática. El archivo `routes/web.php` define las rutas para la interfaz web. A estas rutas se les asigna el grupo de *middleware* `web`, el cual proporciona algunas características como el estado de la sesión y la protección CSRF. Las rutas en `routes/api.php` no tienen estado y se les asigna el grupo de *middleware* `api`.

Para la mayoría de aplicaciones, se comenzará definiendo las rutas en el archivo `routes/web.php`. Se puede acceder a las rutas definidas en `routes/web.php` simplemente escribiendo la URI definida en el navegador. Por ejemplo, se puede acceder a la siguiente ruta simplemente accediendo a `http://tu-app.dev/user` desde el navegador:

    Route::get('/user', 'UserController@index');
    

El `RouteServiceProvider` anidará las rutas definidas en `routes/api.php` bajo un grupo de rutas. Este grupo añadirá el URI `/api` como prefijo de forma automática, por lo que no es necesario definirlo en cada una de las rutas del archivo. Se puede modificar el prefijo y otras opciones del grupo modificando la clase `RouteServiceProvider`.

#### Métodos de rutas disponibles

El *router* permite registrar rutas que respondan a cualquier verbo HTTP:

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);
    

En ocasiones puede ser necesario una ruta que responda a varios verbos HTTP. Puede hacerse utilizando el método `match`. O incluso se puede registrar una ruta que responda a todos los verbos HTTP utilizando el método `any`:

    Route::match(['get', 'post'], '/', function () {
        //
    });
    
    Route::any('foo', function () {
        //
    });
    

#### Protección CSRF

Cualquier formulario HTML que apunte a una ruta `POST`, `PUT` o `DELETE` definida en el archivo de rutas `web` debe incluir un campo con un token CSRF. De lo contrario, la petición se rechazará. Se puede averiguar más sobre protección CSRF en la [documentación CSRF](/docs/{{version}}/csrf):

    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>
    

<a name="redirect-routes"></a>

### Redireccionar rutas

Para definir una ruta que redirecciona a otra URI, se puede utilizar el método `Route::redirect`. Este método evita tener que definir una ruta completa o un controlador para gestionar una simple redirección:

    Route::redirect('/here', '/there', 301);
    

<a name="view-routes"></a>

### Rutas con vistas

Si únicamente se necesita devolver una vista desde una ruta, se puede utilizar el método `Route::view`. Al igual que el método `redirect`, este método es como un acceso directo para no tener que definir la ruta completa o un controlador. El método `view` acepta una URI como primer parámetro y un nombre de vista como segundo. Además, se le puede pasar un array de datos a la vista como tercer parámetro opcional:

    Route::view('/welcome', 'welcome');
    
    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
    

<a name="route-parameters"></a>

## Rutas con parámetros

<a name="required-parameters"></a>

### Parámetros requeridos

En muchas ocasiones es necesario capturar segmentos de la URI dentro de una ruta. Por ejemplo, obtener el id de usuario en la URL. Esto se puede hacer mediante la definición de parámetros de ruta:

    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });
    

Es posible definir tantos parámetros como se requieran en la ruta:

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });
    

Los parámetros de ruta se definen siempre dentro de llaves `{}` y deben contener únicamente caracteres alfabéticos y nunca contener `-` (guiones). Instead of using the `-` character, use an underscore (`_`). Los parámetros de rutas se inyectan directamente en los callback/controladores en orden – los nombres de los argumentos no afectan.

<a name="parameters-optional-parameters"></a>

### Parámetros opcionales

A veces es necesario especificar un parámetro, pero la presencia de este parámetro es opcional. Se puede definir simplemente añadiendo el símbolo `?` detrás del nombre del parámetro. Asegúrese de dar a esa variable un valor por defecto en su callback/controlador:

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });
    
    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });
    

<a name="parameters-regular-expression-constraints"></a>

### Expresiones regulares

Es posible limitar el formato de los elementos dentro de los parámetros de una ruta usando el método `where` en una instancia de *Route*. El método `where` acepta el nombre del parámetro y la expresión regular que define como se debe limitar el parámetro:

    Route::get('user/{name}', function ($name) {
        //
    })->where('name', '[A-Za-z]+');
    
    Route::get('user/{id}', function ($id) {
        //
    })->where('id', '[0-9]+');
    
    Route::get('user/{id}/{name}', function ($id, $name) {
        //
    })->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
    

<a name="parameters-global-constraints"></a>

#### Restricciones globales

Si se desea que un parámetro de ruta siempre este limitado por una expresión regular definida, se puede hace utilizando el método `pattern`. Estos patrones se deben definir en el método `boot` dentro del `RouteServiceProvider`:

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @return void
     */
    public function boot()
    {
        Route::pattern('id', '[0-9]+');
    
        parent::boot();
    }
    

Una vez que el patrón se haya sido definido, se aplicará automáticamente a todas las rutas que utilicen ese nombre como parámetro:

    Route::get('user/{id}', function ($id) {
        // Only executed if {id} is numeric...
    });
    

<a name="named-routes"></a>

## Nombres de rutas

Las rutas con nombre permiten la generación de URL o redirecciones para rutas específicas. Se puede especificar el nombre de una ruta encadenando el método `name` a la definición de la ruta:

    Route::get('user/profile', function () {
        //
    })->name('profile');
    

También se pueden especificar nombres de rutas para acciones de controladores:

    Route::get('user/profile', 'UserController@showProfile')->name('profile');
    

#### Generar URLs con nombres de ruta

Una vez que se ha asignado un nombre a una ruta concreta, se puede utilizar ese nombre para generar URLs o redirecciones a través del *helper* global `route`:

    // Generating URLs...
    $url = route('profile');
    
    // Generating Redirects...
    return redirect()->route('profile');
    

Si la ruta define parámetros, se pueden pasar como segundo argumento a la función `routes`. Los parámetros se insertarán automáticamente en la posición correcta de la URL:

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');
    
    $url = route('profile', ['id' => 1]);
    

#### Inspeccionar la ruta actual

Para determinar si la petición actual coincide con el nombre de alguna ruta, se puede utilizar el método `named` de una instancia de *Route*. Por ejemplo, se puede comprobar el nombre de la ruta actual desde un *middleware* de ruta:

    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->route()->named('profile')) {
            //
        }
    
        return $next($request);
    }
    

<a name="route-groups"></a>

## Grupos de rutas

Los Grupos de Rutas permiten compartir atributos de ruta, por ejemplo *middleware* o *namespaces*, a un grupo de rutas sin necesidad de definir los atributos a cada una de manera individual. Los atributos compartidos se pasan en un como array al primer parámetro del método `Route::group`.

<a name="route-group-middleware"></a>

### Middleware

Para asignar un *middleware* a todas las rutas de un grupo, se puede utilizar el método `middleware` antes de definir el grupo. Los *middleware* se ejecutarán en en el mismo orden que se pasen en el array:

    Route::middleware(['first', 'second'])->group(function () {
        Route::get('/', function () {
            // Uses first & second Middleware
        });
    
        Route::get('user/profile', function () {
            // Uses first & second Middleware
        });
    });
    

<a name="route-group-namespaces"></a>

### *Namespaces*

Otro caso de uso común para los grupos de rutas es el asignar un mismo *namespace* PHP a un grupo de controladores utilizando el método `namespace`:

    Route::namespace('Admin')->group(function () {
        // Controllers Within The "App\Http\Controllers\Admin" Namespace
    });
    

Recordar, el `RouteServiceProvider` incluye ya por defecto los archivos de ruta en un grupo con *namespace*, permitiendo registrar controladores sin tener que especificar el *nemespace* completo `App\Http\Controllers`. Así, únicamente hay que especificar la porción del *namespace* que vendrá tras el *namespace* base `App\Http\Controllers`.

<a name="route-group-sub-domain-routing"></a>

### Rutas de sub-dominios

Los grupos de rutas se pueden utilizar para gestionar el enrutamiento de sub-dominios. A los sub-dominios se les pueden asignar parámetros al igual que a cualquier otra ruta, permitiendo capturaruna porción del sub-dominio para usarla en nuestra ruta o controlador. El sub-dominio se especifica llamando al método `domain` antes de definir el grupo:

    Route::domain('{account}.myapp.com')->group(function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });
    

<a name="route-group-prefixes"></a>

### Prefijos de Rutas

El método `prefix` se puede utilizar para prefijar cada ruta en un grupo con una URI concreta. Por ejemplo, para añadir un prefijo a todas las URIs en el grupo `admin`:

    Route::prefix('admin')->group(function () {
        Route::get('users', function () {
            // Matches The "/admin/users" URL
        });
    });
    

<a name="route-model-binding"></a>

## Enlazar modelos a rutas

Al inyectar el ID de un modelo a una ruta o acción de controlador, normalmente se ejecutará una consulta contra la base de datos para obtener el modelo que corresponde con dicho ID. El *route model binding* ayuda a inyectar instancias de modelo de forma automática en las rutas. Por ejemplo, en lugar de inyectar el ID de usuario, se puede inyectar la instancia de modelo completa `User` que concuerda con el ID dado.

<a name="implicit-binding"></a>

### Enlazado implícito – *Implicit binding*

Laravel resuelve de forma automática los modelos Eloquent que se definen en rutas o acciones de controladores cuyo parámetro coincide con el nombre de un segmento de la ruta. Por ejemplo:

    Route::get('api/users/{user}', function (App\User $user) {
        return $user->email;
    });
    

Puesto que la variable `$user` se requiere como tipo de `App\User` y el nombre de la variable coincide con el segmento URI `{user}`, Laravel inyectará automáticamente la instancia de modelo que corresponda con el valor de la petición URI. Si no se encuentra el modelo en la base de datos, se generará una respuesta HTTP 404 automáticamente.

#### Personalizar el nombre de la clave

Para utilizar una columna alternativa a `id` al obtener el modelo, se puede reemplazar el método `getRouteKeyName` en el modelo Eloquent:

    /**
     * Get the route key for the model.
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }
    

<a name="explicit-binding"></a>

### Enlazado explícito

Para registrar un modelo de forma explícita, utilizar el método `model` para especificar la clase de un parámetro concreto. Se deben definir en el método `boot` del `RouteServiceProvider`:

    public function boot()
    {
        parent::boot();
    
        Route::model('user', App\User::class);
    }
    

A continuación, definir una ruta que contenga un parámetro `{user}`:

    Route::get('profile/{user}', function (App\User $user) {
        //
    });
    

Puesto que se han enlazado todos los parámetros `{user}` al model `App\User`, se inyectará una instancia de `User` en la ruta. Por ejemplo, una petición a `profile/1` inyectará la instancia de `User` desde la base de datos que tenga un ID de `1`.

Si no se encuentra ninguna coincidencia en la base de datos, se retornará una respuesta HTTP 404 de forma automática.

#### Personalizar la lógica de resolución

Para utilizar una lógica propia de resolución, se puede utilizar el método `Route::bind`. La `Closure` que se pasa al método `bind` recibirá el valor del segmento URI y debe retornar la instancia de la clase a inyectar en la ruta:

    public function boot()
    {
        parent::boot();
    
        Route::bind('user', function ($value) {
            return App\User::where('name', $value)->first() ?? abort(404);
        });
    }
    

<a name="form-method-spoofing"></a>

## Falsear el método del formulario

Los formularios HTML no soportan las acciones `PUT`, `PATCH` o `DELETE`. Por lo que al definir rutas `PUT`, `PATCH` o `DELETE` que provengan de un formulario HTML, será necesario añadir un campo oculto (hidden) `_method` en el formulario. El valor del campo `_method` se utilizará como verbo HTTP:

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>
    

Se puede utilizar el *helper* `method_field` para generar el campo `_method`:

    {{ method_field('PUT') }}
    

<a name="accessing-the-current-route"></a>

## Acceder a la ruta actual

Se pueden utilizar los métodos `current`, `currentRouteName` y `currentRouteAction` de la *facade* `Route` para acceder a información sobre la ruta que gestiona la petición actual:

    $route = Route::current();
    
    $name = Route::currentRouteName();
    
    $action = Route::currentRouteAction();
    

Ir a la documentación del API para conocer todos los métodos disponibles de [la clase subyacente de la *facade Route*](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) y [la instancia Route](https://laravel.com/api/{{version}}/Illuminate/Routing/Route.html).