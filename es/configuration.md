# Configuración

- [Introducción](#introduction)
- [Configuración del Entorno](#environment-configuration) 
    - [Obtener la Configuración del Entorno](#retrieving-environment-configuration)
    - [Determinar el Entorno Actual](#determining-the-current-environment)
- [Acceso a Valores de Configuración](#accessing-configuration-values)
- [Configuración de Caché](#configuration-caching)
- [Modo de Mantenimiento](#maintenance-mode)

<a name="introduction"></a>

## Introducción

Todos los archivos de configuración de Laravel se almacenan en el directorio `config`. Cada opción está documentada, por lo que es más que recomendable navegar entre los diferentes archivos y conocer las diferentes opciones.

<a name="environment-configuration"></a>

## Configuración del Entorno

A menudo es útil tener valores de configuración diferentes basados en el entorno de en que la aplicación se ejecuta. Por ejemplo, se puede utilizar un controlador de *cache* local diferente que el que se utiliza en el servidor de producción.

Para hacer esto muy fácil, Laravel utiliza la librería de PHP [DotEnv](https://github.com/vlucas/phpdotenv) de Vance Lucas. En una instalación nueva de Laravel, el directorio raíz de la aplicación contendrá un archivo `.env.example`. Si se instala Laravel través de composer, este archivo automáticamente se renombrará a `.env`. De lo contrario, debe cambiarse manualmente.

Sin embargo, el archivo `.env` no debe incorporarse nunca al repositorio, pues cada desarrollador / servidor que utiliza la aplicación puede requerir una configuración de entorno diferente. Además, esto supondría un riesgo de seguridad en caso de que un intruso acceda a su repositorio, ya que cualquier credencial sensible quedaría expuesta.

Si se desarrolla en conjunto con un equipo, es recomendable seguir incluyendo un archivo `.env.example`. Estableciendo valores de ejemplo en el archivo de configuración, otros desarrolladores del equipo podrán saber cuales son las variables necesarias para ejecutar la aplicación. También puede crear un archivo `.env.testing`. Este archivo reemplazará los valores del archivo `.env` al ejecutar pruebas de *PHPUnit* o al ejecutar comandos *Artisan* con la opción `--env=testing`.

> {tip} Cualquier variable en su archivo `.env` puede ser reemplazada por variables de entorno externas a nivel de servidor o de sistema.

<a name="retrieving-environment-configuration"></a>

### Obtener la Configuración del Entorno

Todas las variables listadas en este archivo se cargarán en la variable super-global de PHP `$_ENV` cada vez que la aplicación reciba una petición. Sin embargo, se puede utilizar el *helper* `env` para recuperar valores de estas variables en sus archivos de configuración. De hecho, revisando los archivos de configuración de Laravel se puede observar que varias de las opciones están usando este *helper*:

    'debug' => env('APP_DEBUG', false),
    

The second value passed to the `env` function is the "default value". This value will be used if no environment variable exists for the given key.

<a name="determining-the-current-environment"></a>

### Determinar el Entorno Actual

The current application environment is determined via the `APP_ENV` variable from your `.env` file. You may access this value via the `environment` method on the `App` [facade](/docs/{{version}}/facades):

    $environment = App::environment();
    

You may also pass arguments to the `environment` method to check if the environment matches a given value. The method will return `true` if the environment matches any of the given values:

    if (App::environment('local')) {
        // The environment is local
    }
    
    if (App::environment(['local', 'staging'])) {
        // The environment is either local OR staging...
    }
    

> {tip} The current application environment detection can be overridden by a server-level `APP_ENV` environment variable. This can be useful when you need to share the same application for different environment configurations, so you can set up a given host to match a given environment in your server's configurations.

<a name="accessing-configuration-values"></a>

## Acceso a Valores de Configuración

You may easily access your configuration values using the global `config` helper function from anywhere in your application. The configuration values may be accessed using "dot" syntax, which includes the name of the file and option you wish to access. A default value may also be specified and will be returned if the configuration option does not exist:

    $value = config('app.timezone');
    

To set configuration values at runtime, pass an array to the `config` helper:

    config(['app.timezone' => 'America/Chicago']);
    

<a name="configuration-caching"></a>

## Configuration Caching

To give your application a speed boost, you should cache all of your configuration files into a single file using the `config:cache` Artisan command. This will combine all of the configuration options for your application into a single file which will be loaded quickly by the framework.

You should typically run the `php artisan config:cache` command as part of your production deployment routine. The command should not be run during local development as configuration options will frequently need to be changed during the course of your application's development.

> {note} If you execute the `config:cache` command during your deployment process, you should be sure that you are only calling the `env` function from within your configuration files.

<a name="maintenance-mode"></a>

## Modo de Mantenimiento

When your application is in maintenance mode, a custom view will be displayed for all requests into your application. This makes it easy to "disable" your application while it is updating or when you are performing maintenance. A maintenance mode check is included in the default middleware stack for your application. If the application is in maintenance mode, a `MaintenanceModeException` will be thrown with a status code of 503.

Para activar el modo de mantenimiento, simplemente ejecutar el comando de Artisan `down`:

    php artisan down
    

You may also provide `message` and `retry` options to the `down` command. The `message` value may be used to display or log a custom message, while the `retry` value will be set as the `Retry-After` HTTP header's value:

    php artisan down --message="Upgrading Database" --retry=60
    

To disable maintenance mode, use the `up` command:

    php artisan up
    

> {tip} Puede personalizar la plantilla del modo de mantenimiento predeterminado definiendo su propia plantilla en `resources/views/errors/503.blade.php`.

#### Modo de Mantenimiento & Colas

Mientras que la aplicación se encuentra en modo de mantenimiento, no se atenderán [colas de trabajo](/docs/{{version}}/queues). Los trabajos continuarán normalmente una vez que la aplicación salga del modo de mantenimiento.

#### Alternativas al Modo de Mantenimiento

Puesto que el modo de mantenimiento requiere que su aplicación tenga varios segundos de tiempo de inactividad, considere alternativas como [Envoyer](https://envoyer.io) para lograr un despliegue de tiempo de inactividad cero con Laravel.