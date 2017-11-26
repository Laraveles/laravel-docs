# Errors & Logging

- [Introducción](#introduction)
- [Configuración](#configuration) 
    - [Detalles de errores](#error-detail)
    - [Almacenamiento del Log](#log-storage)
    - [Niveles de gravedad del Log](#log-severity-levels)
    - [Configuración de *Monolog* personalizada](#custom-monolog-configuration)
- [El gestor de excepciones](#the-exception-handler) 
    - [Report Method](#report-method)
    - [Render Method](#render-method)
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

### Log Storage

De serie, Laravel soporta la escritura de *logs* en archivos `individuales`, archivos `diarios`, el `syslog`, y el `errorlog`. Para configurar qué mecanismo de almacenamiento utiliza Laravel, debe modificar la opción de `log` en su archivo de configuración `config/app.php`. Por ejemplo, si desea utilizar archivos de *log* diarios en lugar de un único archivo, debería establecer el valor de `log` en su archivo de configuración de `app` a `daily`:

    'log' => 'daily'
    

#### Maximum Daily Log Files

When using the `daily` log mode, Laravel will only retain five days of log files by default. If you want to adjust the number of retained files, you may add a `log_max_files` configuration value to your `app` configuration file:

    'log_max_files' => 30
    

<a name="log-severity-levels"></a>

### Log Severity Levels

When using Monolog, log messages may have different levels of severity. By default, Laravel writes all log levels to storage. However, in your production environment, you may wish to configure the minimum severity that should be logged by adding the `log_level` option to your `app.php` configuration file.

Once this option has been configured, Laravel will log all levels greater than or equal to the specified severity. For example, a default `log_level` of `error` will log **error**, **critical**, **alert**, and **emergency** messages:

    'log_level' => env('APP_LOG_LEVEL', 'error'),
    

> {tip} Monolog recognizes the following severity levels - from least severe to most severe: `debug`, `info`, `notice`, `warning`, `error`, `critical`, `alert`, `emergency`.

<a name="custom-monolog-configuration"></a>

### Custom Monolog Configuration

If you would like to have complete control over how Monolog is configured for your application, you may use the application's `configureMonologUsing` method. You should place a call to this method in your `bootstrap/app.php` file right before the `$app` variable is returned by the file:

    $app->configureMonologUsing(function ($monolog) {
        $monolog->pushHandler(...);
    });
    
    return $app;
    

#### Customizing The Channel Name

By default, Monolog is instantiated with name that matches the current environment, such as `production` or `local`. To change this value, add the `log_channel` option to your `app.php` configuration file:

    'log_channel' => env('APP_LOG_CHANNEL', 'my-app-name'),
    

<a name="the-exception-handler"></a>

## The Exception Handler

<a name="report-method"></a>

### The Report Method

All exceptions are handled by the `App\Exceptions\Handler` class. This class contains two methods: `report` and `render`. We'll examine each of these methods in detail. The `report` method is used to log exceptions or send them to an external service like [Bugsnag](https://bugsnag.com) or [Sentry](https://github.com/getsentry/sentry-laravel). By default, the `report` method simply passes the exception to the base class where the exception is logged. However, you are free to log exceptions however you wish.

For example, if you need to report different types of exceptions in different ways, you may use the PHP `instanceof` comparison operator:

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
    

#### The `report` Helper

Sometimes you may need to report an exception but continue handling the current request. The `report` helper function allows you to quickly report an exception using your exception handler's `report` method without rendering an error page:

    public function isValid($value)
    {
        try {
            // Validate the value...
        } catch (Exception $e) {
            report($e);
    
            return false;
        }
    }
    

#### Ignoring Exceptions By Type

The `$dontReport` property of the exception handler contains an array of exception types that will not be logged. For example, exceptions resulting from 404 errors, as well as several other types of errors, are not written to your log files. You may add other exception types to this array as needed:

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

### The Render Method

The `render` method is responsible for converting a given exception into an HTTP response that should be sent back to the browser. By default, the exception is passed to the base class which generates a response for you. However, you are free to check the exception type or return your own custom response:

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

### Reportable & Renderable Exceptions

Instead of type-checking exceptions in the exception handler's `report` and `render` methods, you may define `report` and `render` methods directly on your custom exception. When these methods exist, they will be called automatically by the framework:

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

## HTTP Exceptions

Some exceptions describe HTTP error codes from the server. For example, this may be a "page not found" error (404), an "unauthorized error" (401) or even a developer generated 500 error. In order to generate such a response from anywhere in your application, you may use the `abort` helper:

    abort(404);
    

The `abort` helper will immediately raise an exception which will be rendered by the exception handler. Optionally, you may provide the response text:

    abort(403, 'Unauthorized action.');
    

<a name="custom-http-error-pages"></a>

### Custom HTTP Error Pages

Laravel makes it easy to display custom error pages for various HTTP status codes. For example, if you wish to customize the error page for 404 HTTP status codes, create a `resources/views/errors/404.blade.php`. This file will be served on all 404 errors generated by your application. The views within this directory should be named to match the HTTP status code they correspond to. The `HttpException` instance raised by the `abort` function will be passed to the view as an `$exception` variable:

    <h2>{{ $exception->getMessage() }}</h2>
    

<a name="logging"></a>

## Logging

Laravel provides a simple abstraction layer on top of the powerful [Monolog](https://github.com/seldaek/monolog) library. By default, Laravel is configured to create a log file for your application in the `storage/logs` directory. You may write information to the logs using the `Log` [facade](/docs/{{version}}/facades):

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
    

The logger provides the eight logging levels defined in [RFC 5424](https://tools.ietf.org/html/rfc5424): **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info** and **debug**.

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);
    

#### Contextual Information

An array of contextual data may also be passed to the log methods. This contextual data will be formatted and displayed with the log message:

    Log::info('User failed to login.', ['id' => $user->id]);
    

#### Accessing The Underlying Monolog Instance

Monolog has a variety of additional handlers you may use for logging. If needed, you may access the underlying Monolog instance being used by Laravel:

    $monolog = Log::getMonolog();