# Autenticación

- [Introducción](#introduction) 
    - [Consideraciones de la base de datos](#introduction-database-considerations)
- [Guía Rápida de Autenticación](#authentication-quickstart) 
    - [Rutas](#included-routing)
    - [Vistas](#included-views)
    - [Autenticación](#included-authenticating)
    - [Obtener el usuario autenticado](#retrieving-the-authenticated-user)
    - [Protección de rutas](#protecting-routes)
    - [Intentos de login](#login-throttling)
- [Autenticación manual de usuarios](#authenticating-users) 
    - [Recordar usuarios](#remembering-users)
    - [Otros métodos de autenticación](#other-authentication-methods)
- [Autenticación HTTP básica](#http-basic-authentication) 
    - [Autenticación HTTP basica sin estado](#stateless-http-basic-authentication)
- [Autenticación social](https://github.com/laravel/socialite)
- [Añadir *Guards* personalizados](#adding-custom-guards)
- [Añadir *user providers* personalizados](#adding-custom-user-providers) 
    - [El contrato User Provider](#the-user-provider-contract)
    - [El contrato Authenticatable](#the-authenticatable-contract)
- [Eventos](#events)

<a name="introduction"></a>

## Introducción

> {tip} **¿Quiere comenzar rápido?** Simplemente ejecutar `php artisan make:auth` y `php artisan migrate` una aplicación Laravel. A continuación ya se podrá acceder a `http://tu-app.dev/register`. ¡Estos dos comandos generarán todo el *scaffolding* del sistema de autenticación!

Laravel hace que la implementación de la autenticación sea muy simple. De hecho, prácticamente todo viene configurado de serie. El archivo de configuración de autenticación se encuentra en `config/auth.php`, el cual contiene varias opciones bien documentadas para ajustar el comportamiento de los servicios de autenticación.

Así como el core, las funciones de autenticación de Laravel se basan en "guards" y "providers". Los *guards* definen como se autentican los usuarios para cada petición. Por ejemplo, Laravel incluye un *guard* `session` que mantiene el estado utilizando el almacenamiento de sesión y cookies.

Los *providers* definen como se obtienen los usuarios de la capa de almacenamiento. Laravel incluye soporte para obtener usuarios con Eloquent y el *query builder* de la base de datos. Sin embargo, se pueden definir *providers* adicionales si la aplicación lo necesitara.

¡Todo esto puede sonar confuso ahora! La mayoría de aplicaciones nunca necesitarán modificar la configuración de autenticación por defecto.

<a name="introduction-database-considerations"></a>

### Consideraciones de la base de datos

Por defecto, Laravel incluye un [modelo Eloquent](/docs/{{version}}/eloquent) `App\User` en el directorio `app`. Este modelo se puede utilizar con el driver de autenticación de Eloquent por defecto. Si la aplicación no utiliza Eloquent, se puede especificar el driver `database` que utiliza el *query builder* de Laravel.

Al construir la estructura de base de datos para el modelo `App\User`, hay que comprobar que la columna *password* contiene al menos 60 caracteres de longitud. Mantener la longitud por defecto de esta columna de 255 caracteres sería una buena opción.

También, verificar que la tabla `users` (o equivalente) contiene una columna `remeber_token` de tipo *string* que acepta valores *null* de 100 caracteres. Esta columna se utilizará para almacenar un *token* para los usuarios que seleccionen la opción "recordarme" al identificarse en la aplicación.

<a name="authentication-quickstart"></a>

## Guía rápida de autenticación

Laravel incluye varios controladores pre-desarrollados que se pueden encontrar en el namespace `App\Http\Controllers\Auth`. El controlador `RegisterController` gestiona el registro de nuevos usuarios, `LoginController` gestiona la autenticación, `ForgotPasswordController` se encarga del envío de e-mails para el restablecimiento de contraseñas y `ResetPasswordController` contiene la lógica para restablecer las contraseñas. Cada uno de estos controladores utiliza un *trait* para incluir los métodos necesarios. Para muchas aplicaciones, no será necesario la modificación siquiera de estos controladores.

<a name="included-routing"></a>

### Rutas

Laravel provee una forma rápida de generar todas las rutas y vistas necesarias para la autenticación en un simple comando:

    php artisan make:auth
    

Este comando se debe utilizar en aplicaciones nuevas y instalará un *layout*, vistas de registro y login así como rutas para todos los *end-point* de autenticación. Se generará además un `HomeController` para gestionar las peticiones *post-login* de la aplicación.

<a name="included-views"></a>

### Vistas

Como se ha mencionado anteriormente, el comando `php artisan make:auth` creará todas las vistas necesarias para la autenticación y las ubicará en el directorio `resources/views/auth`.

El comando `make:auth` creará además el directorio `resources/views/layouts` que contiene un *layout* para la aplicación. Todas estas vistas utilizan Bootstrap CSS, pero se puede personalizar al gusto.

<a name="included-authenticating"></a>

### Autenticación

Ahora que se tienen las rutas y vistas configuradas con los controladores de autenticación, se está listo para registrar y autenticar usuarios en la aplicación. Se puede acceder a la aplicación directamente desde el navegador ya que los controladores de autenticación ya tienen la lógica (a través de *traits*) para autenticar usuarios existentes y registrar nuevos en la base de datos.

#### Personalización de rutas

Cuando un usuario se autentica satisfactoriamente, se le redirecciona a la URI `/home`. Se puede personalizar esta redirección post-autenticación definiendo la propiedad `redirectTo` de `LoginController`, `RegisterController` y `ResetPasswordController`:

    protected $redirectTo = '/';
    

Si la ruta de redirección precisa de alguna lógica, se puede definir un método `redirectTo` en lugar de la propiedad `redirectTo`:

    protected function redirectTo()
    {
        return '/path';
    }
    

> {tip} El método `redirectTo` se ejecutará preferiblemente sobre el atributo `redirectTo`.

#### Personalizar el nombre de usuario

Por defecto, Laravel utiliza el campo `email` para la autenticación. Para cambiar esto, se puede definir el método `username` en `LoginController`:

    public function username()
    {
        return 'username';
    }
    

#### Personalización de *guard*

Se puede personalizar el "guard" que se utiliza para autenticar y registrar usuarios. Para comenzar, defina un método `guard` en `LoginController`, `RegisterController` y `ResetPasswordController`. Este método debe retornar una instancia de *Guard*:

    use Illuminate\Support\Facades\Auth;
    
    protected function guard()
    {
        return Auth::guard('guard-name');
    }
    

#### Personalizar la validación/persistencia

Para modificar los campos del formulario que se requieren al registrar un usuario en la aplicación o personalizar cómo se almacenan los usuarios en la base de datos, se puede modificar la clase `RegisterController`. Esta clase es responsable de validar y crear nuevos usuarios en la aplicación.

El método `validator` de `Registercontroller` contiene las reglas de validación para nuevos usuarios. Se pude modificar este método al gusto.

El método `create` de `RegisterController` es responsable de crear un nuevo registro `App\User` en la base de datos utilizando [Eloquent ORM](/docs/{{version}}/eloquent). Se puede modificar este método de acuerdo con las necesidades de la base de datos.

<a name="retrieving-the-authenticated-user"></a>

### Obtener el usuario autenticado

Es posible acceder al usuario autenticado a través de la facade `Auth`:

    use Illuminate\Support\Facades\Auth;
    
    // Get the currently authenticated user...
    $user = Auth::user();
    
    // Get the currently authenticated user's ID...
    $id = Auth::id();
    

Alternativamente, una vez que un usuario está autenticado, se puede acceder al mismo a través de una instancia de `Illuminate\Http\Request`. Recordar, las clases incluidas en la firma del método serán inyectadas:

    <?php
    
    namespace App\Http\Controllers;
    
    use Illuminate\Http\Request;
    
    class ProfileController extends Controller
    {
        /**
         * Update the user's profile.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // $request->user() returns an instance of the authenticated user...
        }
    }
    

#### Determinar si el usuario está autenticado

Para determinar si el usuario ha iniciado sesión en la aplicación, se puede utilizar el método `check` de la facade `Auth`, el cual retornará `true` si el usuario está autenticado:

    use Illuminate\Support\Facades\Auth;
    
    if (Auth::check()) {
        // The user is logged in...
    }
    

> {tip} Incluso si es posible determinar si el usuario está autenticado utilizando el método `check`, normalmente se utilizará un middleware para verificar que el usuario está autenticado antes de permitir el acceso a ciertas rutas o controladores. Para saber más sobre esto puede revisar la documentación en [protección de rutas](/docs/{{version}}/authentication#protecting-routes).

<a name="protecting-routes"></a>

### Protección de rutas

Los [middleware de ruta](/docs/{{version}}/middleware) se pueden utilizar para permitir el acceso a ciertas rutas únicamente a usuarios autenticados. Laravel incluye un *middleware* `auth`, definido en `Illuminate\Auth\Middleware\Authenticate`. Puesto que este *middleware* ya está registrado en el *kernel* HTTP, todo lo que hay que hacer es asignarlo a la definición de una ruta:

    Route::get('profile', function () {
        // Only authenticated users may enter...
    })->middleware('auth');
    

Por supuesto, si se están utilizando [controladores](/docs/{{version}}/controllers), se puede llamar al método `middleware` desde el constructor del controlador en lugar de asignarlo en la definición de la ruta directamente:

    public function __construct()
    {
        $this->middleware('auth');
    }
    

#### Especificar un *Guard*

Al asignar el *middleware* `auth` a una ruta, se puede especificar el *guard* a utilizar para autenticar al usuario. El *guard* especificado corresponderá a una de las claves del *array* `guards` en el archivo de configuración `auth.php`:

    public function __construct()
    {
        $this->middleware('auth:api');
    }
    

<a name="login-throttling"></a>

### Intentos de login

Si está utilizando la clase `LoginController` que incluye Laravel, se incluirá directamente el *trait* `Illuminate\Foundation\Auth\ThrottlesLogins` en el controlador. De forma predeterminada, el usuario no podrá iniciar sesión durante un minuto si no se proporcionan las credenciales correctas después de varios intentos. Este bloqueo es único al nombre de usuario / email y su dirección IP.

<a name="authenticating-users"></a>

## Autenticación manual de usuarios

Por supuesto, no es obligatorio utilizar los controladores de autenticación incluidos en Laravel. Se pueden eliminar y gestionar la autenticación de usuarios utilizando las clases de autenticación de Laravel directamente. No hay que alarmarse, ¡es super sencillo!

Se accede a los servicios de autenticación de Laravel a través de la [facade](/docs/{{version}}/facades) `Auth`, por lo que hay que comprobar que se importa la *facade* `Auth` al comienzo de la clase. A continuación se revisará el método `attempt`:

    <?php
    
    namespace App\Http\Controllers;
    
    use Illuminate\Support\Facades\Auth;
    
    class LoginController extends Controller
    {
        /**
         * Handle an authentication attempt.
         *
         * @return Response
         */
        public function authenticate()
        {
            if (Auth::attempt(['email' => $email, 'password' => $password])) {
                // Authentication passed...
                return redirect()->intended('dashboard');
            }
        }
    }
    

El método `attempt` acepta un *array* de pares clave/valor como primer parámetro. Los valores de este array se utilizarán para encontrar al usuario en la tabla de la base de datos. Así, en el ejemplo anterior, se obtendrá el usuario por el valor de la columna `email`. Si se encuentra el usuario, se comparará la contraseña cifrada almacenada en la base de datos con el valor `password` que se pasó a través del *array*. Nunca se debe cifrar la contraseña del campo `password`, puesto que el framework lo hará automáticamente antes de compararla con el valor cifrado en la base datos. Si las dos contraseñas cifradas coinciden se iniciará una sesión autenticada para el usuario.

El método `attempt` retornará `true` si la autenticación tuvo éxito. De lo contrario, retornará `false`.

El método `intended` redireccionará al usuario a la URL que intentaba acceder antes de ser interceptado por el *middleware* de autenticación. Se puede proporcionar una URI por defecto para cualquier otro caso.

#### Especificar condiciones adicionales

Si se desea, se pueden añadir condiciones extra a la consulta de autenticación además del e-mail y contraseña del usuario. Por ejemplo, se puede verificar que el usuario esté marcado como "activo":

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // The user is active, not suspended, and exists.
    }
    

> {note} En estos ejemplos, `email` noes una opción requerida, sino meramente usado como ejemplo. Usted debe usar el nombre de la columna que corresponda a un "nombre de usuario" en su base de datos.

#### Acceder a instancias *Guard* específicas

Se puede especificar que instancia de *Guard* se ha de utilizar utilizando el método `guard` en la *facade* `Auth`. Esto permite separar la gestión de la autenticación para las diferentes partes de la aplicación utilizando modelos autenticables o tablas de usuarios.

El nombre del *guard* pasado al método `guard` debe corresponder con uno de los *guards* del archivo de configuración `auth.php`:

    if (Auth::guard('admin')->attempt($credentials)) {
        //
    }
    

#### Cerrar sesión – *Log out*

Para desconectar usuarios de la aplicación, se utiliza el método `logout` de la facade `Auth`. Esto borrará la información de autenticación en la sesión del usuario:

    Auth::logout();
    

<a name="remembering-users"></a>

### Recordar usuarios

Para proporcionar a la aplicación de la funcionalidad "recuerdame", se puede pasar un valor booleano como segundo argumento del método `attempt`, el cual mantendrá al usuario autenticado de forma indefinida, o hasta que manualmente se desconecte. Por supuesto, la tabla `users` debe incluir la columna de cadena `remember_token`, la cual será utilizada para almacenar el token "remember me".

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // The user is being remembered...
    }
    

> {tip} Si se está usando el `LoginController` incluido en Laravel, la lógica para "recordar" usuarios ya está incluida en los *traits* que utiliza el controlador.

Si se "recuerdan"usuarios, se puede utilizar el método `viaRemember` para determinar si un usuario fue autenticado a través de la cookie "remember me":

    if (Auth::viaRemember()) {
        //
    }
    

<a name="other-authentication-methods"></a>

### Otros métodos de autenticación

#### Autenticar una instancia de usuario

Para autenticar una instancia existente de un usuario en la aplicación, se puede utilizar el método `login` con ésta. El objeto proporcionado debe ser una implementación del [contrato](/docs/{{version}}/contracts) `Illuminate\Contracts\Auth\Authenticatable`. Por supuesto, el modelo `App\User` proporcionado con Laravel ya implementa esta interfaz:

    Auth::login($user);
    
    // Login and "remember" the given user...
    Auth::login($user, true);
    

Por supuesto, se puede especificar la instancia *guard* que se desee utilizar:

    Auth::guard('admin')->login($user);
    

#### Autenticar un usuario por ID

Para loguear a un usuario en la aplicación por ID, se puede utilizar el método `loginUsingId`. Este método simplemente acepta la clave primaria del usuario que se desea autenticar:

    Auth::loginUsingId(1);
    
    // Login and "remember" the given user...
    Auth::loginUsingId(1, true);
    

#### Autenticar un usuario una vez

Es posible utilizar el método `once` para autenticar a un usuario para una única petición. No se utilizarán sesiones ni *cookies*, por lo que puede ser útil para APIs sin estado:

    if (Auth::once($credentials)) {
        //
    }
    

<a name="http-basic-authentication"></a>

## Autenticación HTTP Basic

La [Autenticación HTTP Basic](https://en.wikipedia.org/wiki/Basic_access_authentication) permite autenticar usuarios en la aplicación de forma sencilla y rápida sin necesidad de una página de "login" dedicada. Para empezar, añadir el [middleware](/docs/{{version}}/middleware) `auth.basic` a la ruta. El middleware `auth.basic` viene de serie con Laravel framework, así que no hay que definirlo:

    Route::get('profile', function () {
        // Only authenticated users may enter...
    })->middleware('auth.basic');
    

Una vez que el *middleware* se ha adjuntado a la ruta, automáticamente se solicitarán las credenciales al acceder a la ruta en el navegador. Por defecto, el *middleware* `auth.basic` utilizará la columna `email` del usuario como "nombre de usuario".

#### Una nota sobre FastCGI

Si se utiliza PHP FastCGI, la autenticación HTTP Basic puede que no funcione correctamente. Se deben añadir las siguientes lineas al archivo `.htaccess`:

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
    

<a name="stateless-http-basic-authentication"></a>

### Autenticación HTTP Basic sin estado

También puede utilizar autenticación HTTP Basic sin establecer una cookie de identificación de usuario en la sesión, lo que resulta particularmente útil para autenticación de API. Para ello, [definir un middleware](/docs/{{version}}/middleware) que llame al método `onceBasic`. Si no se produce una respuesta por el método `onceBasic`, la petición seguirá adelante en la aplicación:

    <?php
    
    namespace Illuminate\Auth\Middleware;
    
    use Illuminate\Support\Facades\Auth;
    
    class AuthenticateOnceWithBasicAuth
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, $next)
        {
            return Auth::onceBasic() ?: $next($request);
        }
    
    }
    

Siguiente, [registrar el middleware de ruta](/docs/{{version}}/middleware#registering-middleware) y fijarlo a una ruta:

    Route::get('api/user', function () {
        // Only authenticated users may enter...
    })->middleware('auth.basic.once');
    

<a name="adding-custom-guards"></a>

## Añadir *Guards* personalizados

Se pueden definir *guards* de autenticación propios utilizando el método `extend` de la *facade* `Auth`. Debe establecer la llamada al método `extend` en un [service provider](/docs/{{version}}/providers). Puesto que Laravel incluye un `AuthServiceProvider`, se puede incluir el código ahí mismo:

    <?php
    
    namespace App\Providers;
    
    use App\Services\Auth\JwtGuard;
    use Illuminate\Support\Facades\Auth;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    
    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();
    
            Auth::extend('jwt', function ($app, $name, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\Guard...
    
                return new JwtGuard(Auth::createUserProvider($config['provider']));
            });
        }
    }
    

Como se puede observar en el ejemplo anterior, el *callback* que se pasa al método `extend` debe retornar una implementación de `Illuminate\Contracts\Auth\Guard`. Esta interfaz contiene los métodos que habrá que implementar para definir un *guard* personalizado. Una vez que se ha definido el *guard*, se puede configurar en la opción `guards</> del archivo de configuración <code>auth.php`:

    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],
    

<a name="adding-custom-user-providers"></a>

## Añadir *user providers* personalizados

Si no se está utilizando una base de datos relacional tradicional para almacenar los usuarios, será necesario extender Laravel con un proveedor de autenticación propio. Utilice el método `provider` de la *facade* `Auth` para definir un *user provider* personalizado:

    <?php
    
    namespace App\Providers;
    
    use Illuminate\Support\Facades\Auth;
    use App\Extensions\RiakUserProvider;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    
    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();
    
            Auth::provider('riak', function ($app, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\UserProvider...
    
                return new RiakUserProvider($app->make('riak.connection'));
            });
        }
    }
    

Una vez que se ha registrado el proveedor utilizando el método `provider`, se puede cambiar a esta implementación en el archivo de configuración `auth.php`. Primero, definir un `provider` que utilice el nuevo driver:

    'providers' => [
        'users' => [
            'driver' => 'riak',
        ],
    ],
    

Después, ya se puede utilizar ese *provider* en la sección `guards` de la configuración:

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    ],
    

<a name="the-user-provider-contract"></a>

### El contrato User Provider

Las implementaciones de `Illuminate\Contracts\Auth\UserProvider` sólo son responsables de sacar una implementación de `Illuminate\Contracts\Auth\Authenticatable` fuera del sistema de almacenamiento, como MySQL, Riak, etc. Estas dos interfaces permiten a los mecanismos de autenticación de Laravel continuar funcionando independientemente de cómo se almacenan los datos del usuario o qué tipo de clase se utiliza para representarlo.

El contrato `Illuminate\Contracts\Auth\UserProvider`:

    <?php
    
    namespace Illuminate\Contracts\Auth;
    
    interface UserProvider {
    
        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);
    
    }
    

La función `retrieveById` por lo general recibe una clave que representa al usuario, tal como un ID auto-incremental de una base de datos MySQL. La implementación de `Authenticatable` que concuerda con el ID se obtendrá y será devuelta por el método.

La función `retrieveByToken` obtiene un usuario por su único `$identifier` y `$token` "remember me", almacenado en el campo `remember_token`. Como el método anterior, ser retornará una implementación de `Authenticatable`.

El método `updateRememberToken` actualiza el campo `remember_token` de `$user` con el nuevo `$token`. El nuevo *token* puede ser un *token* nuevo, asignado a un intento de login satisfactorio "recordarme", o cuando el usuario cierra sesión.

El método `retrieveByCredentials` recibe el *array* de credenciales que se pasan al método `Auth::attempt` cuando se intenta iniciar sesión en una aplicación. Este método es el que debe "consultar" a la capa de almacenamiento por el usuario que coincide con estos credenciales. Por lo general, este método ejecuta una consulta con una condición "where" en `$credentials['username']`. El método debe retornar una implementación de `Authenticatable`. **Este método no debe intentar hacer ninguna validación de contraseñas o autenticación.**

El método `validateCredentials` debe comparar el `$user` dado con los `$credentials` para autenticar al usuario. Por ejemplo, este método debería utilizar `Hash::check` para comparar el valor de `$user->getAuthPassword()` con `$credentials['password']`. Debe retornar `true` o `false` indicando si la contraseña es válida.

<a name="the-authenticatable-contract"></a>

### El contrato Authenticatable

Una vez que ya se conocen de los métodos del `UserProvider`, hay otro contrato para revisar `Authenticatable`. Recordar, el provider debe retornar una implementación de esta interfaz desde los métodos `retriveById` y `retrieveByCredentials`:

    <?php
    
    namespace Illuminate\Contracts\Auth;
    
    interface Authenticatable {
    
        public function getAuthIdentifierName();
        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();
    
    }
    

Esta interfaz es simple. El método `getAuthIdentifierName` debe retornar el nombre del campo "clave primaria" del usuario y `getAuthIdentifier` debe retornar la "clave primaria" del usuario. En una base de MySQL, esto se correspondería con la clave primaria auto-incremental. El método `getAuthPassword` debería devolver la contraseña del usuario cifrada. Esta interfaz permite al sistema de autenticación trabajar con cualquier clase de usuario, independientemente de qué capa de abstracción ORM o almacenamiento de información utiliza. Por defecto, Laravel incluye una clase `User` en la carpeta `app` que implementa esta interfaz, por lo que se puede consultar esta clase para obtener un ejemplo de una implementación.

<a name="events"></a>

## Eventos

Laravel dispara varios [eventos](/docs/{{version}}/events) durante el proceso de autenticación. Se pueden capturar estos eventos estableciendo listeners en el `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Events\Registered' => [
            'App\Listeners\LogRegisteredUser',
        ],
    
        'Illuminate\Auth\Events\Attempting' => [
            'App\Listeners\LogAuthenticationAttempt',
        ],
    
        'Illuminate\Auth\Events\Authenticated' => [
            'App\Listeners\LogAuthenticated',
        ],
    
        'Illuminate\Auth\Events\Login' => [
            'App\Listeners\LogSuccessfulLogin',
        ],
    
        'Illuminate\Auth\Events\Failed' => [
            'App\Listeners\LogFailedLogin',
        ],
    
        'Illuminate\Auth\Events\Logout' => [
            'App\Listeners\LogSuccessfulLogout',
        ],
    
        'Illuminate\Auth\Events\Lockout' => [
            'App\Listeners\LogLockout',
        ],
    
        'Illuminate\Auth\Events\PasswordReset' => [
            'App\Listeners\LogPasswordReset',
        ],
    ];