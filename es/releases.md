# Notas de publicación

- [Esquema de versiones](#versioning-scheme)
- [Política de Soporte](#support-policy)
- [Laravel 5.5](#laravel-5.5)

<a name="versioning-scheme"></a>

## Esquema de versiones

Las versiones de Laravel siguen la siguiente convención: `paradigma.mayor.menor`. Los lanzamientos mayores se producen cada 6 meses (febrero y agosto), mientras que los menores pueden ocurrir varias veces a la semana. Los lanzamientos menores **nunca** contendrán cambios que "rompan" el código.

Al referenciar Laravel framework o sus componentes desde una aplicación o paquete, se debe indicar alguna restricción de versión como `5.5.*`, puesto que los lanzamientos mayores de Laravel incluyen cambios que podrían "romper" el código. Sin embargo, nos esforzamos por que el proceso de actualización entre versiones mayores se pueda realizar en un día o menos.

Los cambios de paradigma se separan por años de diferencia y representan cambios fundamentales en la arquitectura y convenciones del framework. Actualmente, no hay ningún nuevo paradigma en desarrollo.

#### ¿Por qué Laravel no Utiliza un Versionado Semántico?

Por un lado, todos los componentes opcionales de Laravel (Cashier, Dusk, Valet, Socialite, etc.) **si** usan el versionado semántico. Sin embargo, Laravel en sí mismo no lo hace. La razón es que este sistema de versionado semántico es un modo "reduccionista" de determinar si dos piezas de código son compatibles. Incluso cuando se utiliza el versionado semántico, todavía se tiene que instalar el paquete actualizado y ejecutar una batería de tests para estar seguro de que *realmente* nada es incompatible con el código base.

Por el contrario, Laravel utiliza un sistema de versiones más comunicativo con el ámbito actual del lanzamiento. Además, puesto que los lanzamientos menores **nunca** contienen cambios con roturas intencionales, no se debería recibir una actualización con rotura si la restricción de versión sigue la convención `paradigma.mayor.*`.

<a name="support-policy"></a>

## Política de Soporte

Las las versiones LTS –*Long Term Support* (soporte a largo plazo)– como Laravel 5.5, se garantizan 2 años de solución de problemas generales y 3 años de soluciones relativas a seguridad. Estos lanzamientos son los qué más soporte y mantenimiento tienen. Para lanzamientos generales, se solucionarán problemas durante 6 meses y fallos de seguridad durante un año.

<a name="laravel-5.5"></a>

## Laravel 5.5 (LTS)

Laravel 5.5 continua con las mejoras realizadas en Laravel 5.4 añadiendo auto-detección de paquetes, recursos API / transformaciones, auto-registro de comandos de consola, encadenado de *queued jobs*, límites a los *queued jobs*, intentos de ejecución de trabajos limitados por tiempo, e-mails renderizables, excepciones reportables y renderizables, mejor gestión de excepciones, mejoras en las pruebas de bases de datos, reglas de validación personalizadas, recursos front-end para React, los métodos `Route::view` y `Route::redirect`, "bloqueos" para los drivers de caché Memcached y Redis, notificaciones bajo demanda, soporte para Chrome headless en Dusk, atajos para Blade, mejor soporte para proxy y más.

Además, Laravel 5.5 coincide con el lanzamiento de [Laravel Horizon](https://horizon.laravel.com), un nuevo panel de control y sistema de configuración para sus colas basadas en Redis.

> {tip} Esta documentación resume las mejoras más notables del framework; sin embargo, siempre están disponibles registros de cambios más completos [en GitHub](https://github.com/laravel/framework/blob/5.5/CHANGELOG-5.5.md).

### Laravel Horizon

Horizon proporciona un hermoso dashboard y una configuración controlada por código para sus colas Redis de Laravel. Horizon le permite monitorear fácilmente las métricas clave de su sistema de cola, tales como el rendimiento del trabajo, el tiempo de ejecución y las fallas del trabajo.

Toda la configuración se almacena en un único y sencillo archivo de configuración, permitiendo que su configuración permanezca en el lugar del control del codigo donde todo su equipo puede colaborar.

Para obtener más información sobre Horizon, consulte la sección [full Horizon documentation](/docs/{{version}}/horizon)

### Package Discovery

> {video} Hay un [tutorial gratuito en video](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/5) para esta característica disponible en Laracasts.

En versiones anteriores de Laravel, la instalación de un paquete requería normalmente varios pasos adicionales, como añadir el proveedor de servicios a su archivo de configuración `app` y registrar las *facades* relevantes. Sin embargo, a partir de Laravel 5.5, Laravel puede detectar y registrar automáticamente proveedores de servicios y *facades* para usted.

Por ejemplo, puede experimentar esto instalando el popular paquete `barryvdh/laravel-debugbar` en su aplicación Laravel. Una vez instalado el paquete a través de Composer, la barra de depuración estará disponible para su aplicación sin configuración adicional:

    composer require barryvdh/laravel-debugbar
    

Los desarrolladores de paquetes sólo necesitan añadir sus proveedores de servicios y *facades* al archivo `composer. json` del paquete:

    "extra": {
        "laravel": {
            "providers": [
                "Laravel\\Tinker\\TinkerServiceProvider"
            ]
        }
    },
    

Para obtener más información sobre la actualización de sus paquetes para utilizar el proveedor de servicios y el descubrimiento de fachadas, consulte la documentación completa en [desarrollo de paquetes](/docs/{{version}}/packages).

### API Resources

Al crear una API, es posible que necesite una capa de transformación entre sus modelos Eloquent y las respuestas JSON que se devuelven a los usuarios de la aplicación. Las clases de recursos de Laravel le permiten transformar sus modelos y colecciones de modelos en JSON de forma fácil y expresiva.

Una clase de recursos representa un modelo único que necesita ser transformado en una estructura JSON. Por ejemplo, aquí hay una clase de recurso simple `User`:

    <?php
    
    namespace App\Http\Resources;
    
    use Illuminate\Http\Resources\Json\Resource;
    
    class User extends Resource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }
    

Por supuesto, este es sólo el ejemplo más básico de un recurso API. Laravel también proporciona una variedad de métodos para ayudarle a construir sus recursos y colecciones de recursos. Para obtener más información, consulte la [documentación completa](/docs/{{version}}/eloquent-resources) sobre recursos API.

### Comando de consola Auto-Registro

> {video} Hay un [tutorial de vídeo](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/12) gratuito disponible en Laracasts para esta característica.

Cuando cree nuevos comandos de consola, ya no es necesario que los liste manualmente en la lista de la propiedad `$commands` del kernel de Consola. En su lugar, un nuevo método `load` es llamado desde el método `commands` de su kernel, que escaneara el directorio dado en busca de comandos de consola y los registrará automáticamente:

    /**
     * Register the commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');
    
        // ...
    }
    

### Nuevos preajustes de Frontend

> {video} Hay un [tutorial gratuito en video](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/4) para esta característica disponible en Laracasts.

Mientras que el scaffolding de Vue básico todavía está incluido en Laravel 5.5, ahora están disponibles varias nuevas opciones de preajuste del módulo frontal. En una nueva aplicación Laravel, puede intercambiar el scaffolding Vue por un scaffolding React usando el comando `preset`:

    php artisan preset react
    

O bien, puede eliminarlos completamente utilizando la configuración preestablecida `none`. Esta configuración prefijada dejará su aplicación con un archivo Sass basico y algunas utilidades JavaScript simples:

    php artisan preset none
    

> {note} Estos comandos sólo están destinados a ser ejecutados en instalaciones de Laravel nuevas. No deben utilizarse en aplicaciones existentes.

### Cadena de trabajo en cola

La cadena de trabajos permite especificar una lista de los trabajos en cola que deben ejecutarse en secuencia. Si falla un trabajo de la secuencia, el resto de los trabajos no se ejecutarán. Para ejecutar una cadena de trabajos en cola, puede utilizar el método `withChain` en cualquiera de sus trabajos despachables:

    ProvisionServer::withChain([
        new InstallNginx,
        new InstallPhp
    ])->dispatch();
    

### Límite de tasa de trabajos en cola

Si su aplicación interactúa con Redis, ahora puede acelerar sus trabajos en cola por tiempo o concurrencia. Esta característica puede ser de ayuda cuando sus trabajos en cola interactúan con APIs que también tienen una tasa limitada. Por ejemplo, puede acelerar un determinado tipo de trabajo para que sólo se ejecute 10 veces cada 60 segundos:

    Redis::throttle('key')->allow(10)->every(60)->then(function () {
        // Job logic...
    }, function () {
        // Could not obtain lock...
    
        return $this->release(10);
    });
    

> {tip} En el ejemplo anterior, la `key` puede ser cualquier cadena que identifique de forma unívoca el tipo de trabajo que desea clasificar. Por ejemplo, es posible que desee construir la clave basándose en el nombre de la clase del job y los IDs de los modelos Eloquent en los que opera.

Alternativamente, puede especificar el número máximo de *workers* que pueden procesar simultáneamente un trabajo determinado. Esto puede ser útil cuando un trabajo en cola está modificando un recurso que sólo debe ser modificado por un trabajo a la vez. Por ejemplo, podemos limitar los trabajos de un determinado tipo para que sólo sean procesados por un *worker* a la vez:

    Redis::funnel('key')->limit(1)->then(function () {
        // Job logic...
    }, function () {
        // Could not obtain lock...
    
        return $this->release(10);
    });
    

### Intentos de trabajo basados en el tiempo

Como alternativa a definir cuántas veces se puede intentar un trabajo antes de que falle, ahora puede definir un momento en el que el trabajo debería tener tiempo muerto. Esto permite que un trabajo se intente cualquier número de veces dentro de un plazo determinado. Para definir el tiempo en el que un trabajo debe finalizar, añada un método `retryUntil` a la clase de su trabajo:

    /**
     * Determine the time at which the job should timeout.
     *
     * @return \DateTime
     */
    public function retryUntil()
    {
        return now()->addSeconds(5);
    }
    

> {tip} También puede definir un método `retryUntil` en sus *queued event listeners*.

### Objetos de regla de validación

> {video} Hay un [tutorial gratuito en video](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/7) para esta característica disponible en Laracasts.

Los objetos de reglas de validación proporcionan una forma nueva y compacta de añadir reglas de validación personalizadas a su aplicación. En versiones anteriores de Laravel, se utilizó el método `Validator::extend` para añadir reglas de validación personalizadas a través de *Closures*. Sin embargo, esto puede resultar engorroso. En Laravel 5.5, un nuevo comando Artisan `make:rule` generará una nueva regla de validación en el directorio `app/Rules`:

    php artisan make:rule ValidName
    

Un objeto de regla sólo tiene dos métodos: `passes` y `message`. El método `passes` recibe el valor del atributo y el nombre, y retorna `true` o `false` dependiendo si el valor del atributo es válido o no. El método `message` retorna el mensaje de error de la validación que debe ser usado cuando la misma falla:

    <?php
    
    namespace App\Rules;
    
    use Illuminate\Contracts\Validation\Rule;
    
    class ValidName implements Rule
    {
        /**
         * Determine if the validation rule passes.
         *
         * @param  string  $attribute
         * @param  mixed  $value
         * @return bool
         */
        public function passes($attribute, $value)
        {
            return strlen($value) === 6;
        }
    
        /**
         * Get the validation error message.
         *
         * @return string
         */
        public function message()
        {
            return 'The name must be six characters long.';
        }
    }
    

Una vez definida la regla, puede utilizarla simplemente pasando una instancia del objeto de la regla con sus otras reglas de validación:

    use App\Rules\ValidName;
    
    $request->validate([
        'name' => ['required', new ValidName],
    ]);
    

### Trusted Proxy Integration

When running applications behind a load balancer that terminates TLS / SSL certificates, you may notice your application sometimes does not generate HTTPS links. Typically this is because your application is being forwarded traffic from your load balancer on port 80 and does not know it should generate secure links.

To solve this, many Laravel users install the [Trusted Proxies](https://github.com/fideloper/TrustedProxy) package by Chris Fidao. Since this is such a common use case, Chris' package now ships with Laravel 5.5 by default.

A new `App\Http\Middleware\TrustProxies` middleware is included in the default Laravel 5.5 application. This middleware allows you to quickly customize the proxies that should be trusted by your application:

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
        protected $proxies;
    
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
    

### On-Demand Notifications

A veces, es posible que necesite enviar una notificación a alguien que no esté almacenado como "usuario" de su aplicación. Utilizando el nuevo método `Notificación::route`, puede especificar información de enrutamiento de notificación ad-hoc antes de enviar la notificación:

    Notification::route('mail', 'taylor@laravel.com')
                ->route('nexmo', '5555555555')
                ->send(new InvoicePaid($invoice));
    

### Renderable Mailables

> {video} Hay un [tutorial gratuito en video](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/6) para esta característica disponible en Laracasts.

Los Mailables (correos enviados por su aplicación) pueden ahora ser devueltos directamente desde las rutas, permitiéndole una vista previa rápida de los diseños del mailable en el navegador:

    Route::get('/mailable', function () {
        $invoice = App\Invoice::find(1);
    
        return new App\Mail\InvoicePaid($invoice);
    });
    

### Renderable & Reportable Exceptions

> {video} Hay un [tutorial gratuito en vídeo](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/18) disponible en Laracasts para esta característica.

En versiones anteriores de Laravel, es posible que haya tenido que recurrir a la "comprobación de tipo" en su gestor de excepciones para generar una respuesta personalizada para una excepción dada. Por ejemplo, es posible que haya escrito un código como éste en el método `render` de su manejador de excepciones:

    /**
     * Render an exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $exception
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $exception)
    {
        if ($exception instanceof SpecialException) {
            return response(...);
        }
    
        return parent::render($request, $exception);
    }
    

En Laravel 5.5, ahora puede definir un método `render` directamente en sus excepciones. Esto le permite colocar la lógica de renderizado de la respuesta personalizada directamente en la excepción, lo que ayuda a evitar la acumulación de lógica condicional en su manejador de excepciones. Si también desea personalizar la lógica de informes para la excepción, puede definir un método `report` en la clase:

    <?php
    
    namespace App\Exceptions;
    
    use Exception;
    
    class SpecialException extends Exception
    {
        /**
         * Report the exception.
         *
         * @return void
         */
        public function report()
        {
            //
        }
    
        /**
         * Report the exception.
         *
         * @param  \Illuminate\Http\Request
         * @return void
         */
        public function render($request)
        {
            return response(...);
        }
    }
    

### Request Validation

> {video} Hay un [tutorial gratuito en video](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/2) para esta característica disponible en Laracasts.

El objeto `Illuminate\Http\Request` ahora proporciona un método `validate`, lo que le permite validar rápidamente una solicitud entrante desde una ruta *Closure* o controlador:

    use Illuminate\Http\Request;
    
    Route::get('/comment', function (Request $request) {
        $request->validate([
            'title' => 'required|string',
            'body' => 'required|string',
        ]);
    
        // ...
    });
    

### Manejo consistente de excepciones

La gestión de excepciones de validación es ahora coherente en todo el framework. Anteriormente, había varias ubicaciones en el framework que requerían personalización para cambiar el formato predeterminado de las respuestas de error de validación JSON. Por otro lado, el formato predeterminado para las respuestas de validación JSON en Laravel 5.5 ahora se adhiere a la siguiente convención:

    {
        "message": "The given data was invalid.",
        "errors": {
            "field-1": [
                "Error 1",
                "Error 2"
            ],
            "field-2": [
                "Error 1",
                "Error 2"
            ],
        }
    }
    

Todos los formatos de error de validación JSON pueden controlarse definiendo un único método en su clase `App\Exceptions\Handler`. Por ejemplo, la personalización siguiente formateara las respuestas de validación JSON utilizando la convención de Laravel 5.4.

    use Illuminate\Validation\ValidationException;
    
    /**
     * Convert a validation exception into a JSON response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Illuminate\Validation\ValidationException  $exception
     * @return \Illuminate\Http\JsonResponse
     */
    protected function invalidJson($request, ValidationException $exception)
    {
        return response()->json($exception->errors(), $exception->status);
    }
    

### Bloqueos en Cache

Los controladores de caché Redis y Memcached ahora tienen soporte para obtener y liberar "cerraduras"atómicas. Esto proporciona un método simple de obtener cerraduras arbitrarias sin preocuparse por las condiciones de la prueba. Por ejemplo, antes de realizar una tarea, es posible que desee obtener un bloqueo para que ningún otro proceso intente la misma tarea que ya está en curso:

    if (Cache::lock('lock-name', 60)->get()) {
        // Lock obtained for 60 seconds, continue processing...
    
        Cache::lock('lock-name')->release();
    } else {
        // Lock was not able to be obtained...
    }
    

O bien, puede pasar un *Closure* al método `get`. El *Closure* sólo se ejecutará si se puede obtener el bloqueo y éste se liberará automáticamente después de ejecutarse el *Closure*:

    Cache::lock('lock-name', 60)->get(function () {
        // Lock obtained for 60 seconds...
    });
    

Además, puede "bloquear" hasta que la cerradura esté disponible:

    if (Cache::lock('lock-name', 60)->block(10)) {
        // Wait for a maximum of 10 seconds for the lock to become available...
    }
    

### Mejoras en *Blade*

> {video} Hay un [tutorial gratuito en vídeo](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/10) disponible en Laracasts para esta característica.

La programación de una directiva personalizada es a veces más compleja de lo necesario cuando se definen expresiones condicionales simples y personalizadas. Por esta razón, Blade provee un método `Blade::if` que permite definir rápidamente una directiva condicional propia utilizando *Closures*. Por ejemplo, definamos una condición personalizada que compruebe el entorno de aplicación actual. Podemos hacer esto en el método `boot` de nuestro `AppServiceProvider`:

    use Illuminate\Support\Facades\Blade;
    
    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('env', function ($environment) {
            return app()->environment($environment);
        });
    }
    

Una vez que la estructura condicional se ha definido, es muy fácil utilizarla en nuestras plantillas:

    @env('local')
        // The application is in the local environment...
    @else
        // The application is not in the local environment...
    @endenv
    

Además de la capacidad de definir fácilmente directivas condicionales de Blade personalizadas, se han añadido nuevos accesos directos para comprobar rápidamente el estado de autenticación del usuario actual:

    @auth
        // The user is authenticated...
    @endauth
    
    @guest
        // The user is not authenticated...
    @endguest
    

### New Routing Methods

> {video} Hay un [tutorial gratuito en vídeo](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/16) disponible en Laracasts para esta característica.

Si está definiendo una ruta que se redirige a otra URI, ahora puede utilizar el método `Route::redirect`. Este método evita el tener que definir una ruta completa o un controlador para gestionar una simple redirección:

    Route::redirect('/here', '/there', 301);
    

Si únicamente se necesita devolver una vista desde una ruta, se puede utilizar el método `Route::view`. Al igual que el método `redirect`, este método es como un acceso directo para no tener que definir la ruta completa o un controlador. El método `view` acepta una URI como primer parámetro y un nombre de vista como segundo. Además, se le puede pasar un array de datos a la vista como tercer parámetro opcional:

    Route::view('/welcome', 'welcome');
    
    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
    

### Conexiones de Base de Datos "Sticky"

#### La opción `sticky`

Al configurar leer/ escribir las conexiones de la base de datos, una nueva opción de `sticky` configuración está disponible:

    'mysql' => [
        'read' => [
            'host' => '192.168.1.1',
        ],
        'write' => [
            'host' => '196.168.1.2'
        ],
        'sticky'    => true,
        'driver'    => 'mysql',
        'database'  => 'database',
        'username'  => 'root',
        'password'  => '',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_unicode_ci',
        'prefix'    => '',
    ],
    

La opción `sticky` es un valor *optional* que puede ser usado para permitir la lectura inmediata de grabaciones que han sido escritas en la base de datos durante el ciclo de solicitud actual. Si la opción `sticky`está habilitada y se ha realizado una operación de "escritura" contra la base de datos durante el ciclo de solicitud actual, cualquier operación adicional de "lectura" utilizará la conexión de "escritura". Esto garantiza que cualquier dato escrito durante el ciclo de solicitud pueda leerse inmediatamente desde la base de datos durante la misma solicitud. Depende de usted decidir si este es el comportamiento deseado para su aplicación.