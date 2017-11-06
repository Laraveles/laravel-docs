# Sessiones HTTP

- [Introducción](#introduction) 
    - [Configuración](#configuration)
    - [Pre-requisitos del driver](#driver-prerequisites)
- [Usar la sesión](#using-the-session) 
    - [Obtener datos](#retrieving-data)
    - [Almacenar datos](#storing-data)
    - [Datos flash](#flash-data)
    - [Borrar datos](#deleting-data)
    - [Regenerar el ID de sesión](#regenerating-the-session-id)
- [Añadir drivers de sesión personalizados](#adding-custom-session-drivers) 
    - [Implementar el driver](#implementing-the-driver)
    - [Registrar el driver](#registering-the-driver)

<a name="introduction"></a>

## Introducción

Puesto que las aplicaciones HTTP no poseen un estado, las sesiones proveen un modo de almacenar información sobre el usuario a través de las diferentes peticiones. Laravel incluye una gran variedad de sistemas de sesiones a los que se accede a través de un API unificada. Soporte para sistemas populares como [Memcached](https://memcached.org), [Redis](https://redis.io) y bases de datos de serie.

<a name="configuration"></a>

### Configuración

El archivo de configuración de sesión se encuentra en `config/session.php`. Asegúrese de revisar todas las opciones disponibles en este archivo. Por defecto, Laravel esta configurado para utilizar el controlador de sesión `file`, el cual funciona bien para la mayoría de aplicaciones. En aplicaciones en producción, debería considerarse utilizar el controlador `memcached` o `redis` para conseguir un mejor rendimiento.

El la opción de configuración `driver` define el lugar donde se almacenará la información de la sesión para cada petición. Laravel incluye varios drivers:

<div class="content-list">
  <ul>
    <li>
      <code>file</code> - las sesiones se almacenan en <code>storage/framework/sessions/</code>.
    </li>
    <li>
      <code>cookie</code> - las sesiones se almacenan en cookies seguras y encriptadas.
    </li>
    <li>
      <code>database</code> - las sesiones se almacenan en una base de datos relacional.
    </li>
    <li>
      <code>memcached</code> / <code>redis</code> - las sesiones se almacenan en uno de estos veloces almacenamientos basados en cache.
    </li>
    <li>
      <code>array</code> - las sesiones se almacenan en un array PHP y no se persistirán.
    </li>
  </ul>
</div>

> {tip} El driver array se utiliza en [testing](/docs/{{version}}/testing) y previene la persistencia de los datos de sesión.

<a name="driver-prerequisites"></a>

### Pre-requisitos del driver

#### Base de datos

Al utilizar el driver `database` (base de datos), será necesario crear una tabla para almacenar los elementos de la sesión. A continuación se muestra el `Schema` para dicha tabla:

    Schema::create('sessions', function ($table) {
        $table->string('id')->unique();
        $table->unsignedInteger('user_id')->nullable();
        $table->string('ip_address', 45)->nullable();
        $table->text('user_agent')->nullable();
        $table->text('payload');
        $table->integer('last_activity');
    });
    

Se puede utilizar el comando de Artisan `session:table` para generar esta migración:

    php artisan session:table
    
    php artisan migrate
    

#### Redis

Antes de utilizar las sesiones de Laravel con Redis, hay que instalar el paquete `predis/predis` (~1.0) a través de Composer. Se puede configurar la conexión con Redis en el archivo de configuración `database`. En el archivo de configuración `session`, se utiliza la opción `connection` para especificar que conexión de Redis se utilizará para la sesión.

<a name="using-the-session"></a>

## Usar la sesión

<a name="retrieving-data"></a>

### Obtener datos

Hay dos formas de trabajar con los datos de sesión en Laravel: el *helper* `session` y a través de una instancia `Request`. Primero, revisemos el acceso a la sesión a través de una instancia `Request`, la cual se puede incluir como *sugerencia de tipo* en el método del controlador. Recordar, las dependencias de los métodos de un controlador se inyectan automáticamente a través del [service container](/docs/{{version}}/container):

    <?php
    
    namespace App\Http\Controllers;
    
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;
    
    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function show(Request $request, $id)
        {
            $value = $request->session()->get('key');
    
            //
        }
    }
    

Cuando se obtiene un valor de la sesión, se puede pasar un valor por defecto como segundo argumento al método `get`. Este valor por defecto se retornará si la clave especificada no existe en la sesión. También se puede pasar un `Closure` como valor por defecto al método `get` y si la clave solicitada no existe, se ejecutará y retornará su resultado:

    $value = $request->session()->get('key', 'default');
    
    $value = $request->session()->get('key', function () {
        return 'default';
    });
    

#### El *helper* global *session*

También se puede utilizar la función global de PHP `session` para obtener y almacenar datos en la sesión. Al llamar al *helper* `session` con una cadena como único argumento, retornará el valor de esa clave en la sesión. Si se le llama con una pareja de clave / valor, se almacenarán esos valores en la sesión:

    Route::get('home', function () {
        // Retrieve a piece of data from the session...
        $value = session('key');
    
        // Specifying a default value...
        $value = session('key', 'default');
    
        // Store a piece of data in the session...
        session(['key' => 'value']);
    });
    

> {tip} Hay una pequeña diferencia práctica entre utilizar la sesión a través de una instancia *Request* HTTP y utilizar el *helper* `session`. Ambos métodos son [testables](/docs/{{version}}/testing) a través del método `assertSessionHas`, el cual está disponible en todos los *test cases*.

#### Obtener todos los datos de la sesión

Para recuperar toda la información de la sesión, se puede utilizar el método `all`:

    $data = $request->session()->all();
    

#### Determinar si un elemento existe en la sesión

Para determinar si un valor está presente en la sesión, se puede utilizar el método `has`. Este método retornará `true` si el valor está presente o `null` si no lo está:

    if ($request->session()->has('users')) {
        //
    }
    

Para determinar si un valor está presente en la sesión, incluso si el valor es `null`, se puede utilizar el método `exists`. El método `exists` retornará `true` si el valor está presente:

    if ($request->session()->exists('users')) {
        //
    }
    

<a name="storing-data"></a>

### Almacenar datos

Para guardar datos en la sesión, se utiliza normalmente el método `put` o el *helper* `session`:

    // Via a request instance...
    $request->session()->put('key', 'value');
    
    // Via the global helper...
    session(['key' => 'value']);
    

#### Añadir datos a *arrays* en la sesión

El método `push` se utiliza para incluir un nuevo valor en un elemento de la sesión que sea un array. Por ejemplo, si la clave `user.teams` contiene un array de nombres, se puede añadir un nuevo valor al array así:

    $request->session()->push('user.teams', 'developers');
    

#### Obtener & eliminar un elemento

El método `pull` obtendrá y eliminará un elemento de la sesión en un única declaración:

    $value = $request->session()->pull('key', 'default');
    

<a name="flash-data"></a>

### Mostrar informacion

Muchas veces deseara almacenar elementos en la sesión solo para la siguiente petición. Puede hacer esto usando el método `flash`. La información almacenada en sesión utilizando este método solo estará disponible durante la petición HTTP subsecuente y luego sera eliminada. Mostrar informacion es principalmente util para mensajes de corta vida:

    $request->session()->flash('status', 'Task was successful!');
    

Si se necesita mantener la información *flash* durante varias peticiones, se puede utilizar el método `reflash`, el cual mantendrá los datos *flash* para una petición más. Si únicamente se necesita mantener una información concreta, utilizar el método `keep`:

    $request->session()->reflash();
    
    $request->session()->keep(['username', 'email']);
    

<a name="deleting-data"></a>

### Borrar datos

El método `forget` eliminará una porción de información de la sesión. Si desea borrar toda la información, se debe utilizar el método `flush`:

    $request->session()->forget('key');
    
    $request->session()->flush();
    

<a name="regenerating-the-session-id"></a>

### Regenerando el ID de sesion

Regenerar el ID de sesión es una tarea que se suele llevar a cabo para prevenir ataques de [fijación de sesión](https://en.wikipedia.org/wiki/Session_fixation) por parte de usuarios maliciosos en la aplicación.

Laravel regenera el ID de sesión de forma automática durante la autenticación si se está utilizando el `LoginController` que lleva incorporado; sin embargo, se puede hacer de forma manual con el método `regenerate`.

    $request->session()->regenerate();
    

<a name="adding-custom-session-drivers"></a>

## Añadir drivers de sesión personalizados

<a name="implementing-the-driver"></a>

#### Implementar el Driver

Un driver de sesión personalizado debe implementar `SessionHanldlerInterface`. La interfaz contiene unos pocos métodos que se deben implementar. Un ejemplo de una implementación para MongoDB sería algo así:

    <?php
    
    namespace App\Extensions;
    
    class MongoHandler implements SessionHandlerInterface
    {
        public function open($savePath, $sessionName) {}
        public function close() {}
        public function read($sessionId) {}
        public function write($sessionId, $data) {}
        public function destroy($sessionId) {}
        public function gc($lifetime) {}
    }
    

> {tip} Laravel no incluye ningún directorio para almacenar extensiones. Se pueden establecer donde convengan. En este ejemplo, se ha creado un directorio `Extensions` para almacenar el `MongoHandler`.

Puesto que el propósito de estos métodos no es intuitivo, se van a revisar a continuación:

<div class="content-list">
  <ul>
    <li>
      El método <code>open</code> se puede utilizar en sistemas de almacenamiento de sesión basados en archivos. Puesto que Laravel incluye el controlador de sesión <code>file</code>, casi nunca será necesario poner nada en este método. Se puede dejar vacío. Es solo un hecho sobre el pobre diseño de interfaz (del cual discutiremos luego) que PHP requiere para implementar este método.
    </li>
    <li>
      El método <code>close</code>, como el <code>open</code>, puede no tenerse en cuenta. Para muchos de los controladores, este no es necesario.
    </li>
    <li>
      El método <code>read</code> debe retornar la versión de la cadena de la información de sesión asociada con el <code>$sessionId</code> dado. No es necesario hacer ninguna serializacion u otra codificación cuando recupera o almacena información de sesión, ya que Laravel se encargará de esto automáticamente.
    </li>
    <li>
      El método <code>write</code> debe escribir la cadena <code>$data</code> asociada con el <code>$sessionId</code> en algún sistema de almacenamiento persistente, como MongoDB, Dynamo, etc. De nuevo, no se debe realizar ninguna serialización – Laravel lo gestionará por nosotros.
    </li>
    <li>
      El método <code>destroy</code> debe eliminar la información asociada con el <code>$sessionId</code> del almacenamiento.
    </li>
    <li>
      El método <code>gc</code> debe destruir toda la información de sesión que sea anterior al <code>$lifetime</code> dado, el cual es un <em>timestamp</em> de UNIX. Para sistemas auto-expirables, como <em>Memcached</em> y <em>Redis</em>, este método debe dejarse vacío.
    </li>
  </ul>
</div>

<a name="registering-the-driver"></a>

#### Registrar el Driver

Una vez que se ha implementado el driver, ya se puede registrar en el framework. Para añadir drivers de sesión adicionales a Laravel, se puede utilizar el método `extend` de la `Session` [facade](/docs/{{version}}/facades). Se debe llamar al método `extend` desde el método `boot` de un [service provider](/docs/{{version}}/providers). Se podría hacer desde el `AppServiceProvider` existente o crear uno nuevo:

    <?php
    
    namespace App\Providers;
    
    use App\Extensions\MongoSessionStore;
    use Illuminate\Support\Facades\Session;
    use Illuminate\Support\ServiceProvider;
    
    class SessionServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Session::extend('mongo', function ($app) {
                // Return implementation of SessionHandlerInterface...
                return new MongoSessionStore;
            });
        }
    
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }
    

Una vez que el driver de sesión se ha registrado, se puede utilizar el driver `mongo` del archivo de configuración `config/session.php`.