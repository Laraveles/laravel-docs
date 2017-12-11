# Peticiones HTTP

- [Acceder a la petición](#accessing-the-request) 
    - [Método & ruta de la petición](#request-path-and-method)
    - [Peticiones PSR-7](#psr7-requests)
- [*Trimming* de datos & normalización](#input-trimming-and-normalization)
- [Obtener datos de entrada](#retrieving-input) 
    - [Datos de entrada antiguos](#old-input)
    - [Cookies](#cookies)
- [Archivos](#files) 
    - [Recuperación de archivos subidos](#retrieving-uploaded-files)
    - [Almacenamiento de archivos subidos](#storing-uploaded-files)
- [Configuring Trusted Proxies](#configuring-trusted-proxies)

<a name="accessing-the-request"></a>

## Acceder a la petición

Para obtener una instancia de la petición HTTP actual vía inyección de dependencias, se debe hacer *type-hint* de la clase `Illuminate\Http\Request` en el método del controlador. La instancia de la petición entrante será automáticamente inyectada por el [service container](/docs/{{version}}/container):

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
            $name = $request->input('name');
    
            //
        }
    }
    

#### Inyección de dependencias & parámetros de rutas

Si el método del controlador también espera datos de entrada de un parámetro en la ruta se deben listar los parámetros de ruta después de las otras dependencias. Por ejemplo, si la ruta está definida así:

    Route::put('user/{id}', 'UserController@update');
    

Se podría hacer *type-hint* de `Illuminate\Http\Request` y acceder al parámetro de ruta `id` definiendo el método del controlador de la siguiente forma:

    <?php
    
    namespace App\Http\Controllers;
    
    use Illuminate\Http\Request;
    
    class UserController extends Controller
    {
        /**
         * Update the specified user.
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
    

#### Accediendo a las peticiones usando *Closures* de ruta

Se puede usar el *type-hinting* de la clase `Illuminate\Http\Request` en un *Closure* de ruta. El *service container* inyectará automáticamente la petición entrante dentro del *Closure* al ejecutarse:

    use Illuminate\Http\Request;
    
    Route::get('/', function (Request $request) {
        //
    });
    

<a name="request-path-and-method"></a>

### Método & ruta de la petición

La instancia de `Illuminate\Http\Request` provee una variedad de métodos para examinar una petición HTTP dentro de la aplicación, extiende de la clase `Symfony\Component\HttpFoundation\Request`. A continuación se muestran los métodos más importantes.

#### Obtener la ruta de la petición

El método `path` retorna la información de la ruta de la petición. Así que, si la petición se realizara sobre `http://domain.com/foo/bar`, el método `path` retornaría `foo/bar`:

    $uri = $request->path();
    

El método `is` permite verificar si la ruta coincide con un patrón determinado. Se puede utilizar el carácter `*` como comodín al utilizar este método:

    if ($request->is('admin/*')) {
        //
    }
    

#### Obtener la URL de la petición

Para obtener la URL completa de una petición entrante se pueden usar los métodos `url` o `fullUrl`. El método `url` retorna la URL sin la cadena de consulta, mientras que `fullUrl` incluye todos los parámetros:

    // Without Query String...
    $url = $request->url();
    
    // With Query String...
    $url = $request->fullUrl();
    

#### Obtener el método de la petición

El método `method` retornará el verbo HTTP de la petición. Además se puede utilizar el método `isMethod` para verificar que el verbo HTTP coincide con una cadena dada:

    $method = $request->method();
    
    if ($request->isMethod('post')) {
        //
    }
    

<a name="psr7-requests"></a>

### Peticiones PSR-7

El [estándar PSR-7](http://www.php-fig.org/psr/psr-7/) especifica interfaces para los mensajes HTTP, incluyendo respuestas y peticiones. Si se desea obtener una instancia de una petición PSR-7 en lugar de una petición de Laravel, primero se deben instalar algunas librerías. Laravel utiliza el componente *Symfony HTTP Message Bridge* para convertir las típicas peticiones y respuestas de Laravel en implementaciones compatibles con PSR-7:

    composer require symfony/psr-http-message-bridge
    composer require zendframework/zend-diactoros
    

Una vez instaladas las librerías, se puede obtener una petición PSR-7 usando el *type-hinting* de la interfaz de la petición dentro del *route Closure* o del método de un controlador:

    use Psr\Http\Message\ServerRequestInterface;
    
    Route::get('/', function (ServerRequestInterface $request) {
        //
    });
    

> {tip} Si se retorna una instancia de una respuesta PSR-7 desde la ruta o controlador, automáticamente se convierte en una instancia de respuesta de Laravel que se mostrará por el framework.

<a name="input-trimming-and-normalization"></a>

## *Trimming* de datos & normalización

Por defecto, Laravel incluye los *middleware* `TrimStrings` y `ConvertEmptyStringsToNull` de manera global dentro del *stack* de *middlewares*. Estos *middleware* se listan en la clase `App\Http\Kernel`. Los *middleware* aplicarán la función `trim` automáticamente todos los campos de cadena entrantes a petición, así como convertirán cualquier campo de cadena vacío en `null`. Esto le permite no tener que preocuparse por las cuestiones de normalización en sus rutas y controladores.

Si desea deshabilitar este comportamiento, puede eliminar los dos middleware de la pila de *middleware* de su aplicación eliminándolos de la propiedad `$middleware` de su clase `App\Http\Kernel`.

<a name="retrieving-input"></a>

## Obtener datos de entrada

#### Obtener todos los datos de entrada

También se pueden recuperar todos los datos de entrada como un `array` usando el método `all`:

    $input = $request->all();
    

#### Recuperar un valor de entrada

Usando unos pocos métodos sencillos, se puede acceder a todos los datos ingresados por el usuario desde la instancia de `Illuminate\Http\Request` sin preocuparse por el método HTTP que se haya usado para la petición. Sin importar el verbo HTTP, el método `input` se puede usar para recuperar las entradas del usuario:

    $name = $request->input('name');
    

Se puede pasar un valor por defecto como segundo argumento del método `input`. Este valor se retornará si el valor de la entrada solicitada no está presente en la petición:

    $name = $request->input('name', 'Sally');
    

Cuando se trabaja con formularios que contienen *arrays*, se usa la "notación por puntos" o "*dot notation*" para acceder a los datos:

    $name = $request->input('products.0.name');
    
    $names = $request->input('products.*.name');
    

#### Retrieving Input From The Query String

While the `input` method retrieves values from entire request payload (including the query string), the `query` method will only retrieve values from the query string:

    $name = $request->query('name');
    

If the requested query string value data is not present, the second argument to this method will be returned:

    $name = $request->query('name', 'Helen');
    

You may call the `query` method without any arguments in order to retrieve all of the query string values as an associative array:

    $query = $request->query();
    

#### Retrieving Input Via Dynamic Properties

You may also access user input using dynamic properties on the `Illuminate\Http\Request` instance. For example, if one of your application's forms contains a `name` field, you may access the value of the field like so:

    $name = $request->name;
    

When using dynamic properties, Laravel will first look for the parameter's value in the request payload. If it is not present, Laravel will search for the field in the route parameters.

#### Retrieving JSON Input Values

When sending JSON requests to your application, you may access the JSON data via the `input` method as long as the `Content-Type` header of the request is properly set to `application/json`. You may even use "dot" syntax to dig into JSON arrays:

    $name = $request->input('user.name');
    

#### Retrieving A Portion Of The Input Data

If you need to retrieve a subset of the input data, you may use the `only` and `except` methods. Both of these methods accept a single `array` or a dynamic list of arguments:

    $input = $request->only(['username', 'password']);
    
    $input = $request->only('username', 'password');
    
    $input = $request->except(['credit_card']);
    
    $input = $request->except('credit_card');
    

> {tip} The `only` method returns all of the key / value pairs that you request; however, it will not return key / values pairs that are not present on the request.

#### Determining If An Input Value Is Present

You should use the `has` method to determine if a value is present on the request. The `has` method returns `true` if the value is present on the request:

    if ($request->has('name')) {
        //
    }
    

When given an array, the `has` method will determine if all of the specified values are present:

    if ($request->has(['name', 'email'])) {
        //
    }
    

If you would like to determine if a value is present on the request and is not empty, you may use the `filled` method:

    if ($request->filled('name')) {
        //
    }
    

<a name="old-input"></a>

### Old Input

Laravel allows you to keep input from one request during the next request. This feature is particularly useful for re-populating forms after detecting validation errors. However, if you are using Laravel's included [validation features](/docs/{{version}}/validation), it is unlikely you will need to manually use these methods, as some of Laravel's built-in validation facilities will call them automatically.

#### Flashing Input To The Session

The `flash` method on the `Illuminate\Http\Request` class will flash the current input to the [session](/docs/{{version}}/session) so that it is available during the user's next request to the application:

    $request->flash();
    

You may also use the `flashOnly` and `flashExcept` methods to flash a subset of the request data to the session. These methods are useful for keeping sensitive information such as passwords out of the session:

    $request->flashOnly(['username', 'email']);
    
    $request->flashExcept('password');
    

#### Flashing Input Then Redirecting

Since you often will want to flash input to the session and then redirect to the previous page, you may easily chain input flashing onto a redirect using the `withInput` method:

    return redirect('form')->withInput();
    
    return redirect('form')->withInput(
        $request->except('password')
    );
    

#### Retrieving Old Input

To retrieve flashed input from the previous request, use the `old` method on the `Request` instance. The `old` method will pull the previously flashed input data from the [session](/docs/{{version}}/session):

    $username = $request->old('username');
    

Laravel also provides a global `old` helper. If you are displaying old input within a [Blade template](/docs/{{version}}/blade), it is more convenient to use the `old` helper. If no old input exists for the given field, `null` will be returned:

    <input type="text" name="username" value="{{ old('username') }}">
    

<a name="cookies"></a>

### Cookies

#### Retrieving Cookies From Requests

All cookies created by the Laravel framework are encrypted and signed with an authentication code, meaning they will be considered invalid if they have been changed by the client. To retrieve a cookie value from the request, use the `cookie` method on a `Illuminate\Http\Request` instance:

    $value = $request->cookie('name');
    

Alternatively, you may use the `Cookie` facade to access cookie values:

    $value = Cookie::get('name');
    

#### Attaching Cookies To Responses

You may attach a cookie to an outgoing `Illuminate\Http\Response` instance using the `cookie` method. You should pass the name, value, and number of minutes the cookie should be considered valid to this method:

    return response('Hello World')->cookie(
        'name', 'value', $minutes
    );
    

The `cookie` method also accepts a few more arguments which are used less frequently. Generally, these arguments have the same purpose and meaning as the arguments that would be given to PHP's native [setcookie](https://secure.php.net/manual/en/function.setcookie.php) method:

    return response('Hello World')->cookie(
        'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
    );
    

Alternatively, you can use the `Cookie` facade to "queue" cookies for attachment to the outgoing response from your application. The `queue` method accepts a `Cookie` instance or the arguments needed to create a `Cookie` instance. These cookies will be attached to the outgoing response before it is sent to the browser:

    Cookie::queue(Cookie::make('name', 'value', $minutes));
    
    Cookie::queue('name', 'value', $minutes);
    

#### Generating Cookie Instances

If you would like to generate a `Symfony\Component\HttpFoundation\Cookie` instance that can be given to a response instance at a later time, you may use the global `cookie` helper. This cookie will not be sent back to the client unless it is attached to a response instance:

    $cookie = cookie('name', 'value', $minutes);
    
    return response('Hello World')->cookie($cookie);
    

<a name="files"></a>

## Files

<a name="retrieving-uploaded-files"></a>

### Retrieving Uploaded Files

You may access uploaded files from a `Illuminate\Http\Request` instance using the `file` method or using dynamic properties. The `file` method returns an instance of the `Illuminate\Http\UploadedFile` class, which extends the PHP `SplFileInfo` class and provides a variety of methods for interacting with the file:

    $file = $request->file('photo');
    
    $file = $request->photo;
    

You may determine if a file is present on the request using the `hasFile` method:

    if ($request->hasFile('photo')) {
        //
    }
    

#### Validating Successful Uploads

In addition to checking if the file is present, you may verify that there were no problems uploading the file via the `isValid` method:

    if ($request->file('photo')->isValid()) {
        //
    }
    

#### File Paths & Extensions

The `UploadedFile` class also contains methods for accessing the file's fully-qualified path and its extension. The `extension` method will attempt to guess the file's extension based on its contents. This extension may be different from the extension that was supplied by the client:

    $path = $request->photo->path();
    
    $extension = $request->photo->extension();
    

#### Other File Methods

There are a variety of other methods available on `UploadedFile` instances. Check out the [API documentation for the class](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/File/UploadedFile.html) for more information regarding these methods.

<a name="storing-uploaded-files"></a>

### Storing Uploaded Files

To store an uploaded file, you will typically use one of your configured [filesystems](/docs/{{version}}/filesystem). The `UploadedFile` class has a `store` method which will move an uploaded file to one of your disks, which may be a location on your local filesystem or even a cloud storage location like Amazon S3.

The `store` method accepts the path where the file should be stored relative to the filesystem's configured root directory. This path should not contain a file name, since a unique ID will automatically be generated to serve as the file name.

The `store` method also accepts an optional second argument for the name of the disk that should be used to store the file. The method will return the path of the file relative to the disk's root:

    $path = $request->photo->store('images');
    
    $path = $request->photo->store('images', 's3');
    

If you do not want a file name to be automatically generated, you may use the `storeAs` method, which accepts the path, file name, and disk name as its arguments:

    $path = $request->photo->storeAs('images', 'filename.jpg');
    
    $path = $request->photo->storeAs('images', 'filename.jpg', 's3');
    

<a name="configuring-trusted-proxies"></a>

## Configuring Trusted Proxies

When running your applications behind a load balancer that terminates TLS / SSL certificates, you may notice your application sometimes does not generate HTTPS links. Typically this is because your application is being forwarded traffic from your load balancer on port 80 and does not know it should generate secure links.

To solve this, you may use the `App\Http\Middleware\TrustProxies` middleware that is included in your Laravel application, which allows you to quickly customize the load balancers or proxies that should be trusted by your application. Your trusted proxies should be listed as an array on the `$proxies` property of this middleware. In addition to configuring the trusted proxies, you may configure the headers that are being sent by your proxy with information about the original request:

    <?php
    
    namespace App\Http\Middleware;
    
    use Illuminate\Http\Request;
    use Fideloper\Proxy\TrustProxies as Middleware;
    
    class TrustProxies extends Middleware
    {
        /**
         * The trusted proxies for this application.
         *
         * @var array
         */
        protected $proxies = [
            '192.168.1.1',
            '192.168.1.2',
        ];
    
        /**
         * The current proxy header mappings.
         *
         * @var array
         */
        protected $headers = [
            Request::HEADER_FORWARDED => 'FORWARDED',
            Request::HEADER_X_FORWARDED_FOR => 'X_FORWARDED_FOR',
            Request::HEADER_X_FORWARDED_HOST => 'X_FORWARDED_HOST',
            Request::HEADER_X_FORWARDED_PORT => 'X_FORWARDED_PORT',
            Request::HEADER_X_FORWARDED_PROTO => 'X_FORWARDED_PROTO',
        ];
    }
    

#### Trusting All Proxies

If you are using Amazon AWS or another "cloud" load balancer provider, you may not know the IP addresses of your actual balancers. In this case, you may use `**` to trust all proxies:

    /**
     * The trusted proxies for this application.
     *
     * @var array
     */
    protected $proxies = '**';