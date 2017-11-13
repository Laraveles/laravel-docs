# Laravel Socialite

- [Introducción](#introduction)
- [Instalación](#installation)
- [Configuración](#configuration)
- [Rutas](#routing)
- [Parámetros opcionales](#optional-parameters)
- [Ámbitos de acceso – *Access scopes*](#access-scopes)
- [Autenticación sin estado](#stateless-authentication)
- [Obtener los Datos del Usuario](#retrieving-user-details)

<a name="introduction"></a>

## Introducción

Además de la típica autenticación basada en formularios, Laravel proporciona una simple y conveniente forma de autenticar con proveedores OAuth utilizando [Laravel Socialite](https://github.com/laravel/socialite). Socialite actualmente soporta autenticación con Facebook, Twitter, LinkedIn, Google, GitHub y Bitbucket.

> {tip} Adaptadores para otras plataformas se pueden encontrar en [Socialite Providers](https://socialiteproviders.github.io/) gestionado por la comunidad.

<a name="installation"></a>

## Instalación

Para comenzar con Socialite, utilizar Composer para añadir el paquete a las dependencias del proyecto:

    composer require laravel/socialite
    

<a name="configuration"></a>

## Configuración

Antes de usar Socialite, será necesario añadir los credenciales para los servicios OAuth que utilice la aplicación. Estos credenciales se deben colocar en el archivo de configuración `config/services`, y deben utilizar la clave `facebook`, `twitter`, `linkedin`, `google`, `github` o `bitbucket`, en función de los proveedores que requiera la aplicación. Por ejemplo:

    'github' => [
        'client_id' => env('GITHUB_CLIENT_ID'),         // Your GitHub Client ID
        'client_secret' => env('GITHUB_CLIENT_SECRET'), // Your GitHub Client Secret
        'redirect' => 'http://your-callback-url',
    ],
    

> {tip} Si la opción `redirect` contiene una ruta relativa, se resolverá automáticamente a una URL completa.

<a name="routing"></a>

## Rutas

A continuación, ¡ya podemos autenticar usuarios! Habrá que definir dos rutas: una para redireccionar al usuario al proveedor OAuth, y otra como respuesta después de la autenticación del proveedor. Se accederá a Socialite utilizando la *facade* `Socialite`:

    <?php
    
    namespace App\Http\Controllers\Auth;
    
    use Socialite;
    
    class LoginController extends Controller
    {
        /**
         * Redirect the user to the GitHub authentication page.
         *
         * @return \Illuminate\Http\Response
         */
        public function redirectToProvider()
        {
            return Socialite::driver('github')->redirect();
        }
    
        /**
         * Obtain the user information from GitHub.
         *
         * @return \Illuminate\Http\Response
         */
        public function handleProviderCallback()
        {
            $user = Socialite::driver('github')->user();
    
            // $user->token;
        }
    }
    

El método `redirect` se encarga de enviar al usuario al proveedor de OAuth, mientras que el método que `user` leerá la solicitud entrante y recuperará la información del usuario desde el proveedor.

Por supuesto, hay que definir las rutas a los métodos del controlador:

    Route::get('login/github', 'Auth\LoginController@redirectToProvider');
    Route::get('login/github/callback', 'Auth\LoginController@handleProviderCallback');
    

<a name="optional-parameters"></a>

## Parámetros opcionales

Algunos proveedores OAuth soportan parámetros opcionales en la solicitud de redirección. Para incluir cualquier parámetro en la petición, utilizar el método `with` con un array asociativo:

    return Socialite::driver('google')
        ->with(['hd' => 'example.com'])
        ->redirect();
    

> {note} Al usar el método `with`, sea cuidadoso con no pasar ninguna palabra reservada como `state` o `response_type`.

<a name="access-scopes"></a>

## Ámbitos de acceso – *Access scopes*

Antes de redireccionar al usuario, se pueden añadir "ámbitos" (o *scopes*) a la petición utilizando el método `scopes`. Este método juntará los *scopes* existentes con los que se provean nuevos:

    return Socialite::driver('github')
        ->scopes(['read:user', 'public_repo'])
        ->redirect();
    

Se pueden reemplazar todos los *scopes* utilizando el método `setScopes`:

    return Socialite::driver('github')
        ->setScopes(['read:user', 'public_repo'])
        ->redirect();
    

<a name="stateless-authentication"></a>

## Autenticación sin estado

El método `stateless` se puede utilizar para desactivar la verificación de estado de sesión. Es útil cuando se añade la autenticación social a un API:

    return Socialite::driver('google')->stateless()->user();
    

<a name="retrieving-user-details"></a>

## Obtener los datos del usuario

Una vez obtenida la instancia del usuario, se pueden acceder a más detalles sobre el usuario:

    $user = Socialite::driver('github')->user();
    
    // OAuth Two Providers
    $token = $user->token;
    $refreshToken = $user->refreshToken; // not always provided
    $expiresIn = $user->expiresIn;
    
    // OAuth One Providers
    $token = $user->token;
    $tokenSecret = $user->tokenSecret;
    
    // All Providers
    $user->getId();
    $user->getNickname();
    $user->getName();
    $user->getEmail();
    $user->getAvatar();
    

#### Obtener detalles del usuario con un *token*

Si ya se tiene un *token* de acceso válido para un usuario, se pueden obtener sus detalles utilizando el método `userFromToken`:

    $user = Socialite::driver('github')->userFromToken($token);