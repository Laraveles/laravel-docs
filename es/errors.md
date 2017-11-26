# Errores & *Logging*

- [Introducción](#introduction)
- [Configuración](#configuration) 
    - [Detalles de errores](#error-detail)
    - [Almacenamiento de *logs*](#log-storage)
    - [Niveles de gravedad del *log*](#log-severity-levels)
    - [Configuración de *Monolog* personalizada](#custom-monolog-configuration)
- [El gestor de excepciones](#the-exception-handler) 
    - [El método *report*](#report-method)
    - [Método *render*](#render-method)
    - [Excepciones reportables & renderizables](#renderable-exceptions)
- [Excepciones HTTP](#http-exceptions) 
    - [Páginas de error HTTP personalizadas](#custom-http-error-pages)
- [Logging](#logging)

<a name="introduction"></a>

## Introducción

Al comenzar un nuevo proyecto con Laravel, la gestión de errores y excepciones viene ya configurada. La clase `App\Excepciones\Handler` es donde todas las excepciones activadas por su aplicación se registran y se devuelven al usuario. Profundizaremos en esta clase a lo largo de esta documentación.

Para *logging*, Laravel utiliza la biblioteca [Monolog](https://github.com/Seldaek/monolog), que proporciona soporte para una variedad de potentes gestores de *log*. Laravel configura varios de estos gestores, lo que le permite elegir entre un único archivo de *log*, archivos de *log* rotativos o escribir la información de errores en el *log* del sistema.

<a name="configuration"></a>

## Configuración

<a name="error-detail"></a>

### Detalles de errores

La opción `debug` en su archivo de configuración `config/app.php` determina la cantidad de información sobre un error se debe mostrar al usuario. De forma predeterminada, esta opción está configurada para respetar el valor de la variable de entorno `APP_DEBUG`, que se almacena en su archivo `.env`.

Para el desarrollo local, debe establecer la variable `APP_DEBUG` de entorno en `true`. En su entorno de producción, este valor siempre debe ser `false`. Si el valor se ajusta a `true` en producción, corre el riesgo de exponer los valores de configuración sensibles a los usuarios finales de la aplicación.

<a name="log-storage"></a>

### Almacenamiento de *logs*

De serie, Laravel soporta la escritura de *logs* en archivos `individuales`, archivos `diarios`, el `syslog`, y el `errorlog`. Para configurar qué mecanismo de almacenamiento utiliza Laravel, debe modificar la opción de `log` en su archivo de configuración `config/app.php`. Por ejemplo, si desea utilizar archivos de *log* diarios en lugar de un único archivo, debería establecer el valor de `log` en su archivo de configuración de `app` a `daily`:

    'log' => 'daily'
    

#### Máximo de archivos diarios de *log*

Cuando se utiliza el modo de registro `daily`, Laravel sólo conserva cinco días de archivos de *log* por defecto. Si desea ajustar el número de archivos retenidos, puede añadir un valor de configuración `log_max_files` a su archivo de configuración `app`:

    'log_max_files' => 30
    

<a name="log-severity-levels"></a>

### Niveles de gravedad del *log*

Al usar *Monolog*, los mensajes de registro pueden tener diferentes niveles de gravedad. Por defecto, Laravel guarda la información de cualquier nivel de *log*. Sin embargo, en su entorno de producción, es posible que desee configurar la gravedad mínima que se debe registrar añadiendo la opción `log_level` al archivo de configuración `app.php`.

Una vez configurada la opción, Laravel registrará todos los niveles mayores o iguales a la gravedad especificada. Por ejemplo, un `log_nivel` de `error` registrará mensajes de **error**, **críticos**, **alerta** y **emergencia**:

    'log_level' => env('APP_LOG_LEVEL', 'error'),
    

> {tip} Monolog reconoce los siguientes niveles de gravedad - de menos severo a más severo: `debug`, `info`, `notice`, `warning`, `error`, `critical`, `alert`, `emergency`.

<a name="custom-monolog-configuration"></a>

### Configuración de *Monolog* personalizada

Para tener un control completo sobre la configuración de *Monolog*, se puede utilizar el método `configureMonologUsing` de la aplicación. Este método se debe llamar en el archivo `bootstrap/app.php` justo antes de donde el archivo retorna la variable `$app`:

    $app->configureMonologUsing(function ($monolog) {
        $monolog->pushHandler(...);
    });
    
    return $app;
    

#### Personalizar el nombre del canal

Por defecto, *Monolog* se instala con un nombre que coincide con el entorno actual, como `production` o `local`. Para cambiar este valor, añada la opción `log_channel` a su archivo de configuración `app.php`:

    'log_channel' => env('APP_LOG_CHANNEL', 'my-app-name'),
    

<a name="the-exception-handler"></a>

## El gestor de excepciones

<a name="report-method"></a>

### El método *report*

Todas las excepciones se gestionan por la clase `App\Exceptions\Handler`. Esta clase contiene dos métodos: `report` y `render`. Se examinarán en detalle. El método `report` se utiliza para registrar excepciones o enviarlas a un servicio externo como [Bugsnag](https://bugsnag.com) o [Sentry](https://github.com/getsentry/sentry-laravel). Por defecto, el método `report` simplemente pasa la excepción a la clase base donde la excepción se añade al *log*. Sin embargo, se pueden añadir al *log* tantas excepciones como se desee.

Por ejemplo, para reportar diferentes tipos de excepciones de modos diferentes, se puede utilizar el operador de comparación `instanceof`:

    /**
     * Report or log an exception.
     *
     * This is a great spot to send exceptions to Sentry, Bugsnag, etc.
     *
     * @param  \Exception  $exception
     * @return void
     */
    public function report(Exception $exception)
    {
        if ($exception instanceof CustomException) {
            //
        }
    
        return parent::report($exception);
    }
    

#### El helper `report`

A veces es posible que necesite reportar una excepción pero continuar manejando la solicitud actual. El helper `report` le permite notificar rápidamente una excepción utilizando el método `report` del gestor de excepciones sin mostrar una página de error:

    public function isValid($value)
    {
        try {
            // Validate the value...
        } catch (Exception $e) {
            report($e);
    
            return false;
        }
    }
    

#### Ignorar excepciones por tipo

La propiedad `$dontReport` del gestor de excepciones contiene un *array* de excepciones que no deben ser registradas. Por ejemplo, las excepciones resultantes de errores 404, así como otros tipos de errores, no se escriben en los ficheros *log*. Puede agregar otros tipos de excepciones a este *array* según sea necesario:

    /**
     * A list of the exception types that should not be reported.
     *
     * @var array
     */
    protected $dontReport = [
        \Illuminate\Auth\AuthenticationException::class,
        \Illuminate\Auth\Access\AuthorizationException::class,
        \Symfony\Component\HttpKernel\Exception\HttpException::class,
        \Illuminate\Database\Eloquent\ModelNotFoundException::class,
        \Illuminate\Validation\ValidationException::class,
    ];
    

<a name="render-method"></a>

### El método *render*

El método `render` es responsable de convertir una excepción en una respuesta HTTP que debe enviarse al navegador. Por defecto, la excepción se pasa a la clase base que genera una respuesta automática. Sin embargo, se puede comprobar el tipo de la excepción o retornar una respuesta diferente:

    /**
     * Render an exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $exception
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $exception)
    {
        if ($exception instanceof CustomException) {
            return response()->view('errors.custom', [], 500);
        }
    
        return parent::render($request, $exception);
    }
    

<a name="renderable-exceptions"></a>

### Excepciones *reportables* & *renderizables*

En lugar de comprobar el tipo de excepciones en los métodos `report` y `render` del gestor de excepciones, puede definir métodos `report` y `render` directamente en su excepción personalizada. Si estos métodos existen, el framework los llamará automáticamente:

    <?php
    
    namespace App\Exceptions;
    
    use Exception;
    
    class RenderException extends Exception
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
         * Render the exception into an HTTP response.
         *
         * @param  \Illuminate\Http\Request
         * @return \Illuminate\Http\Response
         */
        public function render($request)
        {
            return response(...);
        }
    }
    

<a name="http-exceptions"></a>

## Excepciones HTTP

Algunas excepciones describen códigos de error HTTP desde el servidor. Por ejemplo, un error "page not found" (404), un "unauthorized error" (401) o incluso un error 500 generado manualmente. Para generar una respuesta de este tipo desde cualquier punto de su aplicación, puede utilizar el helper `abort`:

    abort(404);
    

El helper `abort` lanzará una excepción inmediatamente que será procesada por el gestor de excepciones. Opcionalmente, se puede proporcionar el texto de respuesta:

    abort(403, 'Unauthorized action.');
    

<a name="custom-http-error-pages"></a>

### Páginas de error HTTP personalizadas

Laravel facilita la visualización de páginas de error personalizadas para varios códigos de estado HTTP. Por ejemplo, para personalizar la página de error para el código de estado HTTP 404, crear el archivo `resources/views/errors/404.blade.php`. Este archivo se servirá en todos los errores 404 generados por su aplicación. Los nombres de las vistas de este directorio deben coincidir con el código de estado HTTP al que corresponden. La instancia de `HttpException` planteada por la función `abort` pasará a la vista como variable `$exception`:

    <h2>{{ $exception->getMessage() }}</h2>
    

<a name="logging"></a>

## Registro – *Logging*

Laravel proporciona una capa de abstracción simple sobre la potente biblioteca [Monolog](https://github.com/seldaek/monolog). De forma predeterminada, Laravel está configurado para crear un archivo de registro para su aplicación en el directorio `storage/logs`. Se puede escribir información en los *logs* utilizando la [facade](/docs/{{version}}/facades) `Log`:

    <?php
    
    namespace App\Http\Controllers;
    
    use App\User;
    use Illuminate\Support\Facades\Log;
    use App\Http\Controllers\Controller;
    
    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            Log::info('Showing user profile for user: '.$id);
    
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }
    

El *logger* proporciona los ocho niveles de *logging* definidos en [RFC 5424](https://tools.ietf.org/html/rfc5424): **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info** y **debug**.

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);
    

#### Información contextual

Se puede también pasar un *array* con datos contextuales a los métodos de *log*. Los datos contextuales se formatearán y mostrarán con el mensaje de *log*:

    Log::info('User failed to login.', ['id' => $user->id]);
    

#### Acceder a la instancia *Monolog* subyacente

*Monolog* tiene una gran variedad de gestores adicionales que se pueden utilizar para *logging*. Si es necesario, se puede acceder a la instancia de Monolog subyacente que utiliza Laravel:

    $monolog = Log::getMonolog();