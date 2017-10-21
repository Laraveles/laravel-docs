# HTTP Session

- [Introduction](#introduction) 
    - [Configuration](#configuration)
    - [Driver Prerequisites](#driver-prerequisites)
- [Using The Session](#using-the-session) 
    - [Retrieving Data](#retrieving-data)
    - [Storing Data](#storing-data)
    - [Flash Data](#flash-data)
    - [Deleting Data](#deleting-data)
    - [Regenerating The Session ID](#regenerating-the-session-id)
- [Adding Custom Session Drivers](#adding-custom-session-drivers) 
    - [Implementing The Driver](#implementing-the-driver)
    - [Registering The Driver](#registering-the-driver)

<a name="introduction"></a>

## Introduction

Since HTTP driven applications are stateless, sessions provide a way to store information about the user across multiple requests. Laravel ships with a variety of session backends that are accessed through an expressive, unified API. Support for popular backends such as [Memcached](https://memcached.org), [Redis](https://redis.io), and databases is included out of the box.

<a name="configuration"></a>

### Configuration

The session configuration file is stored at `config/session.php`. Be sure to review the options available to you in this file. By default, Laravel is configured to use the `file` session driver, which will work well for many applications. In production applications, you may consider using the `memcached` or `redis` drivers for even faster session performance.

The session `driver` configuration option defines where session data will be stored for each request. Laravel ships with several great drivers out of the box:

<div class="content-list">
  <ul>
    <li>
      <code>file</code> - sessions are stored in <code>storage/framework/sessions</code>.
    </li>
    <li>
      <code>cookie</code> - sessions are stored in secure, encrypted cookies.
    </li>
    <li>
      <code>database</code> - sessions are stored in a relational database.
    </li>
    <li>
      <code>memcached</code> / <code>redis</code> - sessions are stored in one of these fast, cache based stores.
    </li>
    <li>
      <code>array</code> - sessions are stored in a PHP array and will not be persisted.
    </li>
  </ul>
</div>

> {tip} The array driver is used during [testing](/docs/{{version}}/testing) and prevents the data stored in the session from being persisted.

<a name="driver-prerequisites"></a>

### Driver Prerequisites

#### Database

When using the `database` session driver, you will need to create a table to contain the session items. Below is an example `Schema` declaration for the table:

    Schema::create('sessions', function ($table) {
        $table->string('id')->unique();
        $table->unsignedInteger('user_id')->nullable();
        $table->string('ip_address', 45)->nullable();
        $table->text('user_agent')->nullable();
        $table->text('payload');
        $table->integer('last_activity');
    });
    

You may use the `session:table` Artisan command to generate this migration:

    php artisan session:table
    
    php artisan migrate
    

#### Redis

Before using Redis sessions with Laravel, you will need to install the `predis/predis` package (~1.0) via Composer. You may configure your Redis connections in the `database` configuration file. In the `session` configuration file, the `connection` option may be used to specify which Redis connection is used by the session.

<a name="using-the-session"></a>

## Using The Session

<a name="retrieving-data"></a>

### Retrieving Data

There are two primary ways of working with session data in Laravel: the global `session` helper and via a `Request` instance. First, let's look at accessing the session via a `Request` instance, which can be type-hinted on a controller method. Remember, controller method dependencies are automatically injected via the Laravel [service container](/docs/{{version}}/container):

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
    

When you retrieve a value from the session, you may also pass a default value as the second argument to the `get` method. This default value will be returned if the specified key does not exist in the session. If you pass a `Closure` as the default value to the `get` method and the requested key does not exist, the `Closure` will be executed and its result returned:

    $value = $request->session()->get('key', 'default');
    
    $value = $request->session()->get('key', function () {
        return 'default';
    });
    

#### The Global Session Helper

You may also use the global `session` PHP function to retrieve and store data in the session. When the `session` helper is called with a single, string argument, it will return the value of that session key. When the helper is called with an array of key / value pairs, those values will be stored in the session:

    Route::get('home', function () {
        // Retrieve a piece of data from the session...
        $value = session('key');
    
        // Specifying a default value...
        $value = session('key', 'default');
    
        // Store a piece of data in the session...
        session(['key' => 'value']);
    });
    

> {tip} There is little practical difference between using the session via an HTTP request instance versus using the global `session` helper. Both methods are [testable](/docs/{{version}}/testing) via the `assertSessionHas` method which is available in all of your test cases.

#### Retrieving All Session Data

If you would like to retrieve all the data in the session, you may use the `all` method:

    $data = $request->session()->all();
    

#### Determining If An Item Exists In The Session

To determine if a value is present in the session, you may use the `has` method. The `has` method returns `true` if the value is present and is not `null`:

    if ($request->session()->has('users')) {
        //
    }
    

To determine if a value is present in the session, even if its value is `null`, you may use the `exists` method. The `exists` method returns `true` if the value is present:

    if ($request->session()->exists('users')) {
        //
    }
    

<a name="storing-data"></a>

### Storing Data

To store data in the session, you will typically use the `put` method or the `session` helper:

    // Via a request instance...
    $request->session()->put('key', 'value');
    
    // Via the global helper...
    session(['key' => 'value']);
    

#### Pushing To Array Session Values

The `push` method may be used to push a new value onto a session value that is an array. For example, if the `user.teams` key contains an array of team names, you may push a new value onto the array like so:

    $request->session()->push('user.teams', 'developers');
    

#### Retrieving & Deleting An Item

The `pull` method will retrieve and delete an item from the session in a single statement:

    $value = $request->session()->pull('key', 'default');
    

<a name="flash-data"></a>

### Flash Data

Sometimes you may wish to store items in the session only for the next request. You may do so using the `flash` method. Data stored in the session using this method will only be available during the subsequent HTTP request, and then will be deleted. Flash data is primarily useful for short-lived status messages:

    $request->session()->flash('status', 'Task was successful!');
    

If you need to keep your flash data around for several requests, you may use the `reflash` method, which will keep all of the flash data for an additional request. If you only need to keep specific flash data, you may use the `keep` method:

    $request->session()->reflash();
    
    $request->session()->keep(['username', 'email']);
    

<a name="deleting-data"></a>

### Deleting Data

The `forget` method will remove a piece of data from the session. If you would like to remove all data from the session, you may use the `flush` method:

    $request->session()->forget('key');
    
    $request->session()->flush();
    

<a name="regenerating-the-session-id"></a>

### Regenerating The Session ID

Regenerating the session ID is often done in order to prevent malicious users from exploiting a [session fixation](https://en.wikipedia.org/wiki/Session_fixation) attack on your application.

Laravel automatically regenerates the session ID during authentication if you are using the built-in `LoginController`; however, if you need to manually regenerate the session ID, you may use the `regenerate` method.

    $request->session()->regenerate();
    

<a name="adding-custom-session-drivers"></a>

## Adding Custom Session Drivers

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
    

Once the session driver has been registered, you may use the `mongo` driver in your `config/session.php` configuration file.