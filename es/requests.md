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
    

#### Recuperar valores de la cadena de consulta

Mientras que el método `input` recupera valores de toda la carga útil de la solicitud (incluyendo la cadena de consulta), el método `query` sólo recupera valores de la cadena de consulta:

    $name = $request->query('name');
    

Si los datos de valor de la cadena de consulta solicitada no están presentes, se devolverá el segundo argumento de este método:

    $name = $request->query('name', 'Helen');
    

Puede llamar al método `query` sin ningún argumento para recuperar todos los valores de la cadena de consulta como *array* asociativo:

    $query = $request->query();
    

#### Recuperar datos de entrada mediante propiedades dinámicas

También puede acceder a los datos de entrada de usuario utilizando las propiedades dinámicas en la instancia de `Illuminate\Http\Request`. Por ejemplo, si uno de los formularios de su solicitud contiene un campo `name`, puede acceder al valor del campo así:

    $name = $request->name;
    

Cuando se utilizan propiedades dinámicas, Laravel primero buscará el valor del parámetro en la carga útil solicitada. Si no está presente, Laravel buscará el campo en los parámetros de ruta.

#### Recuperar valores de entrada JSON

Cuando envíe solicitudes JSON a su aplicación, puede acceder a los datos JSON mediante el método `input`, siempre que el encabezado `Content-Type` de la solicitud esté correctamente configurado en `application/json`. Incluso puede utilizar la "sintaxis de puntos" para buscar en los *array* JSON:

    $name = $request->input('user.name');
    

#### Recuperar una parte de los datos de entrada

Si se quiere recuperar un subconjunto de los datos de entrada, se pueden usar los métodos `only` y `except`. Ambos métodos aceptan un `array` o una lista dinámica de argumentos:

    $input = $request->only(['username', 'password']);
    
    $input = $request->only('username', 'password');
    
    $input = $request->except(['credit_card']);
    
    $input = $request->except('credit_card');
    

> {tip} El método `only` devuelve todos los pares clave/valor que solicite; sin embargo, no devuelve los pares clave/valores que no están presentes en la solicitud.

#### Determinar si un valor de entrada está presente

Puede utilizar el método `has` para determinar si un valor está presente en la solicitud. El método `has` devuelve `true` si el valor está presente:

    if ($request->has('name')) {
        //
    }
    

Cuando se le pasa un *array*, el método `has` determinará si todos los valores especificados están presentes:

    if ($request->has(['name', 'email'])) {
        //
    }
    

Si desea determinar si un valor está presente en la solicitud y no está vacío, puede utilizar el método `filled`:

    if ($request->filled('name')) {
        //
    }
    

<a name="old-input"></a>

### Datos de entrada antiguos – *Old input*

Laravel le permite mantener los datos de entrada de una solicitud durante la próxima solicitud. Esta característica es particularmente útil para rellenar formularios después de detectar errores de validación. Sin embargo, si está usando la [validación](/docs/{{version}}/validation) incluida en Laravel, es poco probable que tenga que utilizar manualmente estos métodos, ya que algunas de las funciones de validación los llamarán automáticamente.

#### Flashing Input To The Session

El método `flash` en la clase `Illuminate\Http\Request` mantendrá la entrada actual en la [sesión](/docs/{{version}}/session) para que esté disponible durante la próxima solicitud del usuario a la aplicación:

    $request->flash();
    

También puede utilizar los métodos `flashOnly` y `flashExcept` para enviar un subconjunto de los datos de solicitud a la sesión. Estos métodos son útiles para mantener la información confidencial, como las contraseñas, fuera de la sesión:

    $request->flashOnly(['username', 'email']);
    
    $request->flashExcept('password');
    

#### Flashing Input Then Redirecting

Dado que a menudo querrá hacer un *flash input* a la sesión y luego redirigir a la página anterior, puede encadenar fácilmente el *input flashing* a un redireccionamiento usando el método `withInput`:

    return redirect('form')->withInput();
    
    return redirect('form')->withInput(
        $request->except('password')
    );
    

#### Obtener datos de entrada antiguos

Para obtener la entrada de la petición anterior, utilice el método `old` de la instancia `Request`. El método `old` extraerá los datos de entrada anteriores de la [session](/docs/{{version}}/session):

    $username = $request->old('username');
    

Laravel también proporciona un *helper* global `old`. Si está mostrando la entrada antigua dentro de una [plantilla Blade](/docs/{{version}}/blade), es más conveniente utilizar el helper `old`. Si no existe ninguna entrada antigua para el campo dado, se devolverá `null`:

    <input type="text" name="username" value="{{ old('username') }}">
    

<a name="cookies"></a>

### Cookies

#### Obtener las cookies de la petición

Todas las *cookies* creadas por el framework de Laravel están cifradas y firmadas con un código de autenticación, lo que significa que se considerarán inválidas si han sido modificadas por el cliente. Para recuperar un valor de una *cookie* de la solicitud, utilice el método `cookie` en una instancia de `Illuminate\Http\Request`:

    $value = $request->cookie('name');
    

Alternativamente, puede utilizar la *facade* `Cookie` para acceder a los valores de las *cookies*:

    $value = Cookie::get('name');
    

#### Añadir *cookies* a las respuestas

Puede adjuntar una *cookie* a una respuesta con una instancia de `Illuminate\Http\Response` usando el método `cookie`. Debe proporcionar el nombre, valor y número de minutos que la *cookie* debe considerarse válida:

    return response('Hello World')->cookie(
        'name', 'value', $minutes
    );
    

El método `cookie` acepta además ciertos argumentos que se usan menos frecuentemente. Generalmente estos argumentos tienen el mismo propósito y significado que los argumentos del método nativo de PHP [setcookie](https://secure.php.net/manual/en/function.setcookie.php):

    return response('Hello World')->cookie(
        'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
    );
    

Por otro lado, se puede utilizar la *facade* `Cookie` para crear una "cola" de *cookies* a añadir a la respuesta de la aplicación. El método `queue` acepta una instancia de `Cookie` o los argumentos necesarios para crear una instancia de `Cookie`. Estas *cookies* se adjuntarán a la respuesta antes de que se devuelva al navegador:

    Cookie::queue(Cookie::make('name', 'value', $minutes));
    
    Cookie::queue('name', 'value', $minutes);
    

#### Generación de cookies

Si desea generar una instancia de `Symfony\Component\HttpFoundation\Cookie` en una respuesta posterior, puede utilizar el *helper* global `cookie`. Esta *cookie* no será enviada de vuelta al cliente a menos que se adjunte a una instancia de una respuesta:

    $cookie = cookie('name', 'value', $minutes);
    
    return response('Hello World')->cookie($cookie);
    

<a name="files"></a>

## Archivos

<a name="retrieving-uploaded-files"></a>

### Obtener archivos subidos

Puede acceder a los archivos subidos desde una instancia `Illuminate\Http\Request` usando el método `file` o usando propiedades dinámicas. El método `file` devuelve una instancia de la clase `Illuminate\Http\UploadedFile`, que hereda la clase PHP `SplFileInfo` y proporciona una variedad de métodos para interactuar con el archivo:

    $file = $request->file('photo');
    
    $file = $request->photo;
    

Puede determinar si un archivo está presente en la solicitud utilizando el método `hasFile`:

    if ($request->hasFile('photo')) {
        //
    }
    

#### Validación de subidas exitosas

Además de comprobar si el archivo está presente, puede verificar que no hubo problemas para cargar el archivo a través del método `isValid`:

    if ($request->file('photo')->isValid()) {
        //
    }
    

#### Extensiones & rutas de archivo

La clase `UploadedFile` también contiene métodos para acceder a la ruta totalmente calificada del archivo y su extensión. El método `extension` intentará adivinar la extensión del archivo en función de su contenido. Esta extensión puede ser diferente de la que fue suministrada por el cliente:

    $path = $request->photo->path();
    
    $extension = $request->photo->extension();
    

#### Otros métodos

Existen otros métodos en la instancia de `UploadedFile`. Consulte la [documentación API de la clase](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/File/UploadedFile.html) para obtener más información sobre estos métodos.

<a name="storing-uploaded-files"></a>

### Almacenar archivos subidos

Para almacenar un archivo subido, normalmente utilizará uno de sus [filesystems](/docs/{{version}}/filesystem) configurados. La clase `UploadedFile` tiene un método `store` que moverá un archivo subido a uno de sus discos, que puede ser una ubicación en su sistema de archivos local o incluso una ubicación de almacenamiento en la nube como Amazon S3.

El método `store` acepta la ruta donde se debe almacenar el archivo en relación al directorio raíz configurado del sistema de archivos. Esta ruta no debe contener un nombre de archivo, ya que se generará automáticamente un ID único para que sirva como nombre de archivo.

El método `store` también acepta un segundo argumento opcional para el nombre del disco que debe usarse para almacenar el archivo. El método devolverá la ruta del archivo relativa a la raíz del disco:

    $path = $request->photo->store('images');
    
    $path = $request->photo->store('images', 's3');
    

Si no desea que se genere automáticamente un nombre de archivo, puede utilizar el método `storeAs`, que acepta la ruta, el nombre de archivo y el nombre del disco como argumentos:

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