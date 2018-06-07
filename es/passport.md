# Autenticación API (Passport)

- [Introducción](#introduction)
- [Instalación](#installation) 
    - [Inicio rápido Frontend](#frontend-quickstart)
    - [Despliegue de *Passport*](#deploying-passport)
- [Configuración](#configuration) 
    - [Duración de token](#token-lifetimes)
- [Emisión de tokens de acceso](#issuing-access-tokens) 
    - [Gestión de clientes](#managing-clients)
    - [Solicitud de tokens](#requesting-tokens)
    - [Refrescando Tokens](#refreshing-tokens)
- [Tokens de concesión de contraseña](#password-grant-tokens) 
    - [Crear un cliente de concesión de contraseña](#creating-a-password-grant-client)
    - [Solicitud de Tokens](#requesting-password-grant-tokens)
    - [Solicitud de todos los ámbitos](#requesting-all-scopes)
- [Fichas de donaciones implícitas](#implicit-grant-tokens)
- [Tokens de concesión de credenciales de cliente](#client-credentials-grant-tokens)
- [Tokens de acceso personal](#personal-access-tokens) 
    - [Crear un cliente de acceso personal](#creating-a-personal-access-client)
    - [Administrar tokens de acceso personal](#managing-personal-access-tokens)
- [Protección de Rutas](#protecting-routes) 
    - [A través de *middleware*](#via-middleware)
    - [Pasando el token de acceso](#passing-the-access-token)
- [Alcance Token](#token-scopes) 
    - [Definiendo *scopes*](#defining-scopes)
    - [Asignación de *scopes* a tokens](#assigning-scopes-to-tokens)
    - [Verificación de *scopes*](#checking-scopes)
- [Consumir su API con JavaScript](#consuming-your-api-with-javascript)
- [Eventos](#events)
- [Testing](#testing)

<a name="introduction"></a>

## Introducción

Laravel ya facilita la autenticación a través de los formularios de inicio de sesión tradicionales, pero ¿qué pasa con las APIs? Las APIs generalmente usan tokens para autenticar usuarios y no mantienen el estado de la sesión entre las solicitudes. Laravel hace que la autenticación de API sea muy sencilla utilizando Laravel Passport, que proporciona una implementación completa del servidor OAuth2 para su aplicación en cuestión de minutos. Passport está construido sobre el servidor [League OAuth2](https://github.com/thephpleague/oauth2-server) que es mantenido por Alex Bilbie.

> {note} Esta documentación asume que usted ya está familiarizado con OAuth2. Si usted no sabe nada acerca de OAuth2, considere familiarizarse con la terminología y características generales de OAuth2 antes de continuar.

<a name="installation"></a>

## Instalación

Para empezar, instale Passport a través del gestor de paquetes Composer:

    composer require laravel/passport
    

El proveedor de servicio Passport registra su propio directorio de migración de base de datos con el framework, por lo que debe migrar su base de datos después de registrar el proveedor. Las migraciones de Passport crearán las tablas que su aplicación necesita para almacenar clientes y tokens de acceso:

    php artisan migrate
    

> {note} Si no va a utilizar las migraciones predeterminadas de Passport, debe llamar al método `Passport::ignoreMigrations` en el método `register` de su `AppServiceProvider`. Puede exportar las migraciones predeterminadas utilizando `php artisan vendor:publish --tag=passport-migrations`.

A continuación, debe ejecutar el comando `passport:install`. Este comando creará las claves de cifrado necesarias para generar tokens de acceso seguro. Además, el comando creará clientes "de acceso personal" y "de concesión de contraseña" que se utilizarán para generar tokens de acceso:

    php artisan passport:install
    

Después de ejecutar el comando, agregue el *trait* `Laravel\Passport\HasApiTokens` en su modelo `App\User`. Este *trait* proporcionará algunos *helpers* a su modelo que le permitirán inspeccionar el token y alcances del usuario autenticado:

    <?php
    
    namespace App;
    
    use Laravel\Passport\HasApiTokens;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    
    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;
    }
    

A continuación, debe llamar al método `Passport::routes` dentro del método `boot` de su `AuthServiceProvider`. Este método registrará las rutas necesarias para emitir tokens de acceso y revocará los tokens de acceso, clientes y tokens de acceso personal:

    <?php
    
    namespace App\Providers;
    
    use Laravel\Passport\Passport;
    use Illuminate\Support\Facades\Gate;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    
    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * The policy mappings for the application.
         *
         * @var array
         */
        protected $policies = [
            'App\Model' => 'App\Policies\ModelPolicy',
        ];
    
        /**
         * Register any authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();
    
            Passport::routes();
        }
    }
    

Por último, en el archivo de configuracion `config/auth.php`, debe cambiar la opción `driver` de `api` a `passport`. Esto le indicará a su aplicación el uso de Passport's `TokenGuard` al autenticar las solicitudes de API entrantes:

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    
        'api' => [
            'driver' => 'passport',
            'provider' => 'users',
        ],
    ],
    

<a name="frontend-quickstart"></a>

### Inicio rápido Frontend

> {note} Para poder utilizar los componentes Passport de Vue, debe estar utilizando el framework JavaScript [Vue](https://vuejs.org). Estos componentes también utilizan el framework CSS Bootstrap. Sin embargo, incluso si no está utilizando estas herramientas, los componentes sirven como una referencia valiosa para su propia implementación frontend.

Passport se entrega con una API JSON que puede utilizar para permitir a sus usuarios crear clientes y tokens de acceso personal. Sin embargo, puede llevar mucho tiempo codificar un frontend para interactuar con estas APIs. Por lo tanto, Passport también incluye componentes [Vue](https://vuejs.org) preconstruidos que puede utilizar como ejemplo de implementación o punto de partida para su propia implementación.

Para publicar los componentes Vue de Passport, utilice el comando Artisan `vendor:publish`:

    php artisan vendor:publish --tag=passport-components
    

Los componentes publicados se colocarán en el directorio `resources/assets/js/components`. Una vez que los componentes se hayan publicado, debe registrarlos en su archivo `resources/assets/js/app.js`:

    Vue.component(
        'passport-clients',
        require('./components/passport/Clients.vue')
    );
    
    Vue.component(
        'passport-authorized-clients',
        require('./components/passport/AuthorizedClients.vue')
    );
    
    Vue.component(
        'passport-personal-access-tokens',
        require('./components/passport/PersonalAccessTokens.vue')
    );
    

Después de registrar los componentes, asegúrese de ejecutar `npm run dev` para recompilar sus recursos. Una vez que haya recompilado sus recursos, puede colocar los componentes en una de las plantillas de su aplicación para empezar a crear clientes y tokens de acceso personal:

    <passport-clients></passport-clients>
    <passport-authorized-clients></passport-authorized-clients>
    <passport-personal-access-tokens></passport-personal-access-tokens>
    

<a name="deploying-passport"></a>

### Despliegue de *Passport*

Al implementar Passport en sus servidores de producción por primera vez, es probable que tenga que ejecutar el comando `passport:keys`. Este comando genera las claves de encriptación que necesita Passport para generar el token de acceso. Las claves generadas no suelen mantenerse en el control de origen:

    php artisan passport:keys
    

<a name="configuration"></a>

## Configuración

<a name="token-lifetimes"></a>

### Duración de token

De forma predeterminada, Passport emite tokens de acceso de larga duración que nunca necesitan actualizarse. Si desea configurar una vida útil más corta, puede utilizar los métodos `tokensExpireIn` y `refreshTokensExpireIn`. Estos métodos deben ser llamados desde el método `boot` de su `AuthServiceProvider`:

    use Carbon\Carbon;
    
    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();
    
        Passport::routes();
    
        Passport::tokensExpireIn(Carbon::now()->addDays(15));
    
        Passport::refreshTokensExpireIn(Carbon::now()->addDays(30));
    }
    

<a name="issuing-access-tokens"></a>

## Emisión de tokens de acceso

El uso de OAuth2 con códigos de autorización es como la mayoría de los desarrolladores que están familiarizados con OAuth2 lo usan. Cuando se utilizan códigos de autorización, una aplicación cliente redirigirá a un usuario a su servidor donde aprobará o denegará la solicitud de emisión de un token de acceso al cliente.

<a name="managing-clients"></a>

### Gestión de clientes

En primer lugar, los desarrolladores que necesitan interactuar con la API de su aplicación necesitarán registrar su aplicación con la suya creando un "cliente". Normalmente, esto consiste en proporcionar el nombre de su aplicación y una URL a la que su aplicación puede redirigir después de que los usuarios aprueben su solicitud de autorización.

#### El comando `passport:client`

La forma más sencilla de crear un cliente es utilizando el comando Artisan `passport:client`. Este comando puede ser utilizado para crear sus propios clientes para probar su funcionalidad OAuth2. Cuando ejecute el comando `client`, Passport le pedirá más información sobre su cliente y le proporcionará un ID de cliente y un valor secreto:

    php artisan passport:client
    

#### API JSON

Dado que sus usuarios no podrán utilizar el comando `client`, Passport proporciona una API JSON que puede utilizar para crear clientes. Esto le ahorra la molestia de tener que codificar manualmente los controladores para crear, actualizar y eliminar clientes.

Sin embargo, usted necesitará emparejar la API JSON de Passport con su propio frontend para proporcionar un panel de control para que sus usuarios administren sus clientes. A continuación, revisaremos todos los endpoints de API para administrar clientes. Para mayor comodidad, utilizaremos [Axios](https://github.com/mzabriskie/axios) para demostrar cómo realizar solicitudes HTTP en los puntos finales.

> {tip} Si no desea implementar el frontend de administración de cliente completo usted mismo, puede utilizar el [frontend quickstart](#frontend-quickstart) para tener un frontend completamente funcional en cuestión de minutos.

#### `GET /oauth/clients`

Esta ruta devuelve todos los clientes para el usuario autenticado. Esto es principalmente útil para listar todos los clientes del usuario para que puedan editarlos o borrarlos:

    axios.get('/oauth/clients')
        .then(response => {
            console.log(response.data);
        });
    

#### `POST /oauth/clients`

Esta ruta se utiliza para crear nuevos clientes. Requiere dos tipos de datos: el nombre del cliente `name` y una URL de redirección `redirect`. La URL de redirección `redirect` es donde el usuario será redirigido después de aprobar o rechazar una solicitud de autorización.

Cuando se crea un cliente, se emitirá un ID y un valor secreto. Estos valores se utilizarán cuando solicite tokens de acceso desde su aplicación. La ruta de creación del cliente devolverá la nueva instancia del cliente:

    const data = {
        name: 'Client Name',
        redirect: 'http://example.com/callback'
    };
    
    axios.post('/oauth/clients', data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // List errors on response...
        });
    

#### `PUT /oauth/clients/{client-id}`

Esta ruta se utiliza para actualizar clientes. Requiere dos tipos de datos: el nombre del cliente `name` y una URL de redirección `redirect`. La URL de redirección `redirect` es donde el usuario será redirigido después de aprobar o rechazar una solicitud de autorización. La ruta devolverá la instancia de cliente actualizada:

    const data = {
        name: 'New Client Name',
        redirect: 'http://example.com/callback'
    };
    
    axios.put('/oauth/clients/' + clientId, data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // List errors on response...
        });
    

#### `DELETE /oauth/clients/{client-id}`

Esta ruta se utiliza para eliminar clientes:

    axios.delete('/oauth/clients/' + clientId)
        .then(response => {
            //
        });
    

<a name="requesting-tokens"></a>

### Solicitud de tokens

#### Redireccionando para autenticación

Una vez que se ha creado un cliente, los desarrolladores pueden utilizar su ID de cliente y el valor secreto para solicitar un código de autorización y un token de acceso desde su aplicación. En primer lugar, la aplicación consumidora debe hacer una solicitud de redireccionamiento a la ruta `/oauth/authorize` de su aplicación así:

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => '',
        ]);
    
        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });
    

> {tip} Recuerde, la ruta `/oauth/authorize` ya está definida por el método `Pasaporte::route`. No es necesario definir manualmente esta ruta.

#### Aprobar la solicitud

Al recibir solicitudes de autorización, Passport mostrará automáticamente una plantilla al usuario para que éste pueda aprobar o denegar la solicitud de autorización. Si se aprueba la solicitud, será redirigido de vuelta al `redirect_uri` especificado por la aplicación consumidora. El `redirect_uri` debe coincidir con la URL de redirección especificada `redirect` de cuando se creó el cliente.

Si desea personalizar la pantalla de autorización, puede publicar las vistas de Passport utilizando el comando Artisan `vendor:publish`. Las vistas publicadas se colocarán en `resources/views/vendor/passport`:

    php artisan vendor:publish --tag=passport-views
    

#### Conversión de códigos de autorización para acceder a los Tokens

Si el usuario autoriza la solicitud, se redireccionará a la aplicación consumidora. El consumidor debe entonces enviar una solicitud `POST` a la aplicación para solicitar un token de acceso. La solicitud debe incluir el código de autorización emitido por la aplicación. En este ejemplo, usaremos la librería Guzzle HTTP para hacer la petición `POST`:

    Route::get('/callback', function (Request $request) {
        $http = new GuzzleHttp\Client;
    
        $response = $http->post('http://your-app.com/oauth/token', [
            'form_params' => [
                'grant_type' => 'authorization_code',
                'client_id' => 'client-id',
                'client_secret' => 'client-secret',
                'redirect_uri' => 'http://example.com/callback',
                'code' => $request->code,
            ],
        ]);
    
        return json_decode((string) $response->getBody(), true);
    });
    

Esta ruta `/oauth/token` devuelve una respuesta JSON que contiene los atributos `access_token`, `refresh_token`, y `expires_in`. El atributo `expires_in` contiene el número de segundos hasta que expira el token de acceso.

> {tip} Como la ruta `/oauth/authorize`, la ruta `/oauth/token` se define por el método `Passport::routes`. No es necesario definir manualmente esta ruta.

<a name="refreshing-tokens"></a>

### Refrescando Tokens

Si su aplicación emite tokens de acceso de corta duración, los usuarios tendrán que refrescar sus tokens mediante el token de actualización que se les proporcionó cuando se emitió el token de acceso. En este ejemplo, usaremos la librería Guzzle HTTP para hacer la petición POST:

    $http = new GuzzleHttp\Client;
    
    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'refresh_token',
            'refresh_token' => 'the-refresh-token',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'scope' => '',
        ],
    ]);
    
    return json_decode((string) $response->getBody(), true);
    

La ruta`/oauth/token` devuelve una respuesta JSON que contiene los atributos `access_token`, `refresh_token`, y `expires_in`. El atributo `expires_in` contiene el número de segundos hasta que expira el token de acceso.

<a name="password-grant-tokens"></a>

## Tokens de concesión de contraseña

La concesión de la contraseña OAuth2 permite a sus otros clientes, como una aplicación móvil, obtener una clave de acceso utilizando una dirección de correo electrónico / nombre de usuario y contraseña. Esto le permite emitir tokens de acceso de forma segura a sus clientes de primera mano sin requerir que sus usuarios pasen por todo el flujo de redireccionamiento del código de autorización de OAuth2.

<a name="creating-a-password-grant-client"></a>

### Crear un cliente de concesión de contraseña

Antes de que su aplicación pueda emitir tokens a través de la concesión de contraseña, deberá crear un cliente apropiado para ello. Puede hacerlo utilizando el comando `passport:client` con la opción `--password`. Si ya ha ejecutado el comando `passport:install`, no necesita ejecutar este comando:

    php artisan passport:client --password
    

<a name="requesting-password-grant-tokens"></a>

### Solicitud de tokens

Una vez que haya creado un cliente de concesión de contraseña, puede solicitar un token de acceso enviando una solicitud `POST` a la ruta `/oauth/token` con la dirección de correo electrónico y contraseña del usuario. Recuerde, esta ruta ya está registrada por el método `Passport::routes` para que no sea necesario definirla manualmente. Si la solicitud tiene éxito, recibirá un `access_token` y `refresh_token` en la respuesta JSON del servidor:

    $http = new GuzzleHttp\Client;
    
    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'password',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'username' => 'taylor@laravel.com',
            'password' => 'my-password',
            'scope' => '',
        ],
    ]);
    
    return json_decode((string) $response->getBody(), true);
    

> {tip} Recuerde, los tokens de acceso son de larga duración por defecto. Sin embargo, usted es libre de [configurar la vida útil máxima del token de acceso](#configuration) si es necesario.

<a name="requesting-all-scopes"></a>

### Solicitud de todos los ámbitos

Cuando utilice la concesión de contraseña, puede que desee autorizar el token para todos los ámbitos soportados por su aplicación. Puede hacerlo solicitando el *scope* `*`. Si usted solicita el scope `*`, el método `can` en la instancia del token siempre regresará `true`. Este alcance sólo puede asignarse a un token que se emita utilizando la concesión de `password`:

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'password',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'username' => 'taylor@laravel.com',
            'password' => 'my-password',
            'scope' => '*',
        ],
    ]);
    

<a name="implicit-grant-tokens"></a>

## Implicit Grant Tokens

La concesión implícita es similar a la concesión del código de autorización; sin embargo, el token se devuelve al cliente sin cambiar un código de autorización. Esta concesión se utiliza más comúnmente para aplicaciones JavaScript o móviles en las que las credenciales del cliente no se pueden almacenar de forma segura. Para habilitar la concesión, se debe llamar al método `enableImplicitGrant` en su `AuthServiceProvider`:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();
    
        Passport::routes();
    
        Passport::enableImplicitGrant();
    }
    

Una vez que se ha habilitado una concesión, los desarrolladores pueden utilizar su ID de cliente para solicitar un token de acceso desde su aplicación. La aplicación consumidora debería hacer una petición de redireccionamiento a la ruta `/oauth/authorize` de su aplicación:

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'token',
            'scope' => '',
        ]);
    
        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });
    

> {tip} Recuerde, la ruta `/oauth/authorize` ya está definida por el método `Passport::routes`. No es necesario definir manualmente esta ruta.

<a name="client-credentials-grant-tokens"></a>

## Tokens de concesión de credenciales de cliente

La concesión de credenciales de cliente es adecuada para la autenticación máquina a máquina. Por ejemplo, puede utilizar esta concesión en un trabajo programado que esté realizando tareas de mantenimiento a través de una API. Para usar este método primero necesita agregar un nuevo middleware a su `$routeMiddleware` en `app/Http/Kernel.php`:

    use Laravel\Passport\Http\Middleware\CheckClientCredentials;
    
    protected $routeMiddleware = [
        'client' => CheckClientCredentials::class,
    ];
    

A continuación, conecte este middleware a una ruta:

    Route::get('/user', function(Request $request) {
        ...
    })->middleware('client');
    

Para recuperar un token, haga una solicitud al endpoint `oauth/token`:

    $guzzle = new GuzzleHttp\Client;
    
    $response = $guzzle->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'client_credentials',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'scope' => 'your-scope',
        ],
    ]);
    
    return json_decode((string) $response->getBody(), true)['access_token'];
    

<a name="personal-access-tokens"></a>

## Tokens de acceso personal

A veces, sus usuarios pueden querer emitir tokens de acceso para ellos mismos sin pasar por el típico flujo de redireccionamiento de código de autorización. Permitir a los usuarios emitir tokens para sí mismos a través de la interfaz de usuario de su aplicación puede ser útil para permitir a los usuarios experimentar con su API o puede servir como un enfoque más simple para emitir tokens de acceso en general.

> {note} Los tokens de acceso personal son siempre duraderos. Su vida útil no se modifica cuando se utilizan los métodos `tokensExpireIn` o `refreshTokensExpireIn`.

<a name="creating-a-personal-access-client"></a>

### Crear un cliente de acceso personal

Antes de que su aplicación pueda emitir tokens de acceso personal, deberá crear un cliente de acceso personal. Puede hacerlo utilizando el comando `passport:client` con la opción `--personal`. Si ya ha ejecutado el comando `passport:install`, no necesita ejecutar este comando:

    php artisan passport:client --personal
    

<a name="managing-personal-access-tokens"></a>

### Administrar tokens de acceso personal

Una vez que haya creado un cliente de acceso personal, puede emitir tokens para un usuario determinado utilizando el método `createToken` en la instancia de modelo de `User`. El método `createToken` acepta el nombre del token como primer argumento y una matriz opcional de [scopes](#token-scopes) como segundo argumento:

    $user = App\User::find(1);
    
    // Creating a token without scopes...
    $token = $user->createToken('Token Name')->accessToken;
    
    // Creating a token with scopes...
    $token = $user->createToken('My Token', ['place-orders'])->accessToken;
    

#### API JSON

Passport también incluye una API JSON para la gestión de tokens de acceso personal. Usted puede asociar esto con su propio frontend para ofrecer a sus usuarios un panel de control para gestionar los tokens de acceso personal. A continuación, revisaremos todos los endpoints de la API para administrar los tokens de acceso personal. Para mayor comodidad, utilizaremos [Axios](https://github.com/mzabriskie/axios) para demostrar cómo realizar solicitudes HTTP en los puntos finales.

> {tip} Si no desea implementar el frontend de acceso personal usted mismo, puede utilizar el [quickstart del frontend](#frontend-quickstart) para tener un frontend totalmente funcional en cuestión de minutos.

#### `GET /oauth/scopes`

Esta ruta devuelve todos los [alcances (scopes)](#token-scopes) definidos para su aplicación. Puede utilizar esta ruta para listar los alcances que un usuario puede asignar a un token de acceso personal:

    axios.get('/oauth/scopes')
        .then(response => {
            console.log(response.data);
        });
    

#### `GET /oauth/personal-access-tokens`

Esta ruta devuelve todos los tokens de acceso personal creados por el usuario autenticado. Esto es principalmente útil para listar todos los tokens del usuario para que puedan editarlos o borrarlos:

    axios.get('/oauth/personal-access-tokens')
        .then(response => {
            console.log(response.data);
        });
    

#### `POST /oauth/personal-access-tokens`

Esta ruta crea nuevos tokens de acceso personal. Requiere dos piezas de datos: `name` y `scopes` que deben asignarse al token:

    const data = {
        name: 'Token Name',
        scopes: []
    };
    
    axios.post('/oauth/personal-access-tokens', data)
        .then(response => {
            console.log(response.data.accessToken);
        })
        .catch (response => {
            // List errors on response...
        });
    

#### `DELETE /oauth/personal-access-tokens/{token-id}`

Esta ruta se puede utilizar para borrar los tokens de acceso personal:

    axios.delete('/oauth/personal-access-tokens/' + tokenId);
    

<a name="protecting-routes"></a>

## Protección de Rutas

<a name="via-middleware"></a>

### A través de *middleware*

Passport incluye un [authentication guard (protector de autenticación)](/docs/{{version}}/authentication#adding-custom-guards) que validará los tokens de acceso en las solicitudes entrantes. Una vez que haya configurado el `api` guard para usar el controlador de `passport`, sólo necesita especificar el middleware `auth:api` en cualquier ruta que requiera un token de acceso válido:

    Route::get('/user', function () {
        //
    })->middleware('auth:api');
    

<a name="passing-the-access-token"></a>

### Pasando el token de acceso

Al llamar a rutas protegidas por Passport, los consumidores de la API de su aplicación deben especificar su token de acceso como `Bearer` token en el encabezado `Authorization` de su solicitud. Por ejemplo, cuando se utiliza la biblioteca Guzzle HTTP:

    $response = $client->request('GET', '/api/user', [
        'headers' => [
            'Accept' => 'application/json',
            'Authorization' => 'Bearer '.$accessToken,
        ],
    ]);
    

<a name="token-scopes"></a>

## Alcance Token

<a name="defining-scopes"></a>

### Definiendo *scopes*

Los scopes (alcances) permiten a sus clientes API obtener un conjunto específico de permisos cuando solicitan autorización para acceder a una cuenta. Por ejemplo, si está creando una aplicación de comercio electrónico, no todos los consumidores de la API necesitarán poder realizar pedidos. En su lugar, puede permitir que los consumidores sólo soliciten autorización para acceder al estatus de envío de pedidos. En otras palabras, los alcances permiten a los usuarios de su aplicación limitar las acciones que una aplicación de terceros puede realizar en su nombre.

Puede definir los alcances de su API usando el método `Passport::tokensCan` en el método `boot` de su `AuthServiceProvider`. El método `tokensCan` acepta una matriz de nombres y descripciones de alcance. La descripción del alcance puede ser cualquier cosa que desee y se mostrará a los usuarios en la pantalla de autorización:

    use Laravel\Passport\Passport;
    
    Passport::tokensCan([
        'place-orders' => 'Place orders',
        'check-status' => 'Check order status',
    ]);
    

<a name="assigning-scopes-to-tokens"></a>

### Asignación de *scopes* a tokens

#### Cuando solicita códigos de autorización

Cuando solicita un token de acceso utilizando la concesión de código de autorización, los consumidores deben especificar sus alcances deseados como parámetro de cadena de consulta de `scope`. El parámetro `scope` debe ser una lista de alcances delimitada por el espacio:

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => 'place-orders check-status',
        ]);
    
        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });
    

#### Cuando se emitan Tokens de acceso personal

Si está emitiendo tokens de acceso personal utilizando el método `createToken` del modelo `User`, puede pasar la matriz de alcances deseados como segundo argumento al método:

    $token = $user->createToken('My Token', ['place-orders'])->accessToken;
    

<a name="checking-scopes"></a>

### Verificación de *scopes*

Passport incluye dos middleware que se pueden utilizar para verificar que una solicitud entrante se autentica con un token al que se le ha concedido un alcance dado. Para empezar, agregue el siguiente middleware a la propiedad `$routeMiddleware` de su archivo `app/Http/Kernel.php`:

    'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
    'scope' => \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,
    

#### Verifique todos los alcances

El middleware `scopes` puede ser asignado a una ruta para verificar que el token de acceso de la solicitud entrante tiene *todos* los alcances listados:

    Route::get('/orders', function () {
        // Access token has both "check-status" and "place-orders" scopes...
    })->middleware('scopes:check-status,place-orders');
    

#### Verifique cualquier alcance

El middleware `scope` puede asignarse a una ruta para verificar que el token de acceso de la solicitud entrante tenga *al menos uno* de los alcances listados:

    Route::get('/orders', function () {
        // Access token has either "check-status" or "place-orders" scope...
    })->middleware('scope:check-status,place-orders');
    

#### Comprobación de ámbitos en una instancia de un Token

Una vez que una solicitud autenticada de token de acceso ha entrado en su aplicación, puede comprobar si el token tiene un alcance determinado utilizando el método `tokenCan` en la instancia de `Usuario` autenticado:

    use Illuminate\Http\Request;
    
    Route::get('/orders', function (Request $request) {
        if ($request->user()->tokenCan('place-orders')) {
            //
        }
    });
    

<a name="consuming-your-api-with-javascript"></a>

## Consumir su API con JavaScript

Al crear una API, puede ser extremadamente útil poder consumir su propia API desde su aplicación JavaScript. Este enfoque al desarrollo de API permite que su propia aplicación consuma la misma API que usted está compartiendo con el mundo. La misma API puede ser consumida por su aplicación web, aplicaciones móviles, aplicaciones de terceros y cualquier SDK que pueda publicar en varios administradores de paquetes.

Normalmente, si desea consumir su API desde su aplicación JavaScript, debería enviar manualmente un token de acceso a la aplicación y pasarlo con cada solicitud. Sin embargo, Passport incluye un middleware que puede manejar esto por usted. Todo lo que necesita hacer es añadir el middleware `CreateFreshApiToken` al grupo de middleware `web`:

    'web' => [
        // Other middleware...
        \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
    ],
    

Este middleware Passport adjuntará una cookie `laravel_token` a sus respuestas salientes. Esta cookie contiene un JWT cifrado que Passport utilizará para autenticar las solicitudes API de su aplicación JavaScript. Ahora, puede realizar solicitudes a la API de su aplicación sin pasar explícitamente un token de acceso:

    axios.get('/api/user')
        .then(response => {
            console.log(response.data);
        });
    

Cuando se utiliza este método de autenticación, el scaffolding JavaScript predeterminado de Laravel instruye a Axios para que siempre envíe en el encabezado el `X-CSRF-TOKEN` y el `X-Requested-With`. Sin embargo, debe asegurarse de incluir su token CSRF en un [meta tag HTML](/docs/{{version}}/csrf#csrf-x-csrf-token):

    window.axios.defaults.headers.common = {
        'X-Requested-With': 'XMLHttpRequest',
    };
    

> {note} Si está utilizando un framework JavaScript diferente, debe asegurarse de que está configurado para enviar en los encabezados el `X-CSRF-TOKEN` y `X-Requested-With` con cada solicitud saliente.

<a name="events"></a>

## Eventos

Passport levanta eventos cuando se emiten tokens de acceso y tokens de actualización. Usted puede usar estos eventos para limpiar o revocar otros tokens de acceso en su base de datos. Puede adjuntar oyentes a estos eventos en el `EventServiceProvider` de su aplicación:

```php
/**
 * The event listener mappings for the application.
 *
 * @var array
 */
protected $listen = [
    'Laravel\Passport\Events\AccessTokenCreated' => [
        'App\Listeners\RevokeOldTokens',
    ],

    'Laravel\Passport\Events\RefreshTokenCreated' => [
        'App\Listeners\PruneOldTokens',
    ],
];
```

<a name="testing"></a>

## Testing

El método `actingAs` de Passport se puede utilizar para especificar el usuario actualmente autenticado, así como sus alcances. El primer argumento dado al método `actingAs` es la instancia de usuario y el segundo es un array de alcances que debe ser otorgado al token del usuario:

    public function testServerCreation()
    {
        Passport::actingAs(
            factory(User::class)->create(),
            ['create-servers']
        );
    
        $response = $this->post('/api/create-server');
    
        $response->assertStatus(200);
    }