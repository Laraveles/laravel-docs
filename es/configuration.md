# Configuration

- [Introduction](#introduction)
- [Configuración del entorno](#environment-configuration) 
    - [Obtener la configuración del entorno](#retrieving-environment-configuration)
    - [Determinar el entorno actual](#determining-the-current-environment)
- [Acceso a los valores de configuración](#accessing-configuration-values)
- [Configuración de almacenamiento caché](#configuration-caching)
- [Modo mantenimiento](#maintenance-mode)

<a name="introduction"></a>

## Introduction

Todos los archivos de configuración del *framework* Laravel están almacenados en el directorio `config`. Cada opción está documentada, así que no dude en consultar los archivos y familiarizarse con las diferentes opciones disponibles.

<a name="environment-configuration"></a>

## Configuración del entorno

A menudo es útil tener diferentes valores de configuración basados en el entorno donde se ejecute la aplicación. Por ejemplo, usted puede desear utilizar un majeador de *cache* diferente local diferente que el que se utiliza en su servidor de producción.

Para hacer esto más fácil, Laravel utiliza la librería de PHP [DotEnv](https://github.com/vlucas/phpdotenv) de Vance Lucas. En una instalación nueva de Laravel, el directorio raíz de la aplicación contendrá un archivo `.env.example`. Si usted instala Laravel través de *Composer*, este archivo automáticamente será renombrado como `.env`. De lo contrario, usted debe renombrarlo manualmente.

Su fichero `.env` no debería ser incorporado al control de versiones de su aplicación, pues cada desarrollador / servidor que utiliza la aplicación podría requerir una configuración de entorno diferente. Además, esto supondría un riesgo de seguridad en caso de que un intruso acceda a su repositorio de control de versiones, ya que cualquier credencial sensible quedaría expuesta.

Si se desarrolla en conjunto con un equipo, es recomendable seguir incluyendo un archivo `.env.example`. Estableciendo valores de ejemplo en el archivo de configuración, otros desarrolladores del equipo podrán saber cuales son las variables necesarias para ejecutar la aplicación. También puede crear un archivo `.env.testing`. Este archivo reemplazará los valores del archivo `.env` al ejecutar pruebas de *PHPUnit* o al ejecutar comandos *Artisan* con la opción `--env=testing`.

> {tip} Cualquier variable en su archivo `.env` puede ser reemplazada por variables de entorno externas a nivel de servidor o de sistema.

<a name="retrieving-environment-configuration"></a>

### Obtener la Configuración del Entorno

Todas las variables listadas en este archivo se cargarán en la variable super-global de PHP `$_ENV` cada vez que la aplicación reciba una petición. Sin embargo, se puede utilizar el *helper* `env` para recuperar valores de estas variables en sus archivos de configuración. De hecho, revisando los archivos de configuración de Laravel se puede observar que varias de las opciones están usando este *helper*:

    'debug' => env('APP_DEBUG', false),
    

El segundo valor pasado a la función `env` es el "valor por defecto". Este valor se utilizará si no existe ninguna variable de entorno para la clave dada.

<a name="determining-the-current-environment"></a>

### Determinar el Entorno Actual

El entorno actual de la aplicación se determina mediante la variable `APP_ENV` del archivo `.env`. Se puede acceder a este valor mediante el método `environment` en la [facade](/docs/{{version}}/facades) `App`:

    $environment = App::environment();
    

También se pueden pasar argumentos al método `environment` para comprobar si el entorno corresponde a un valor determinado. El método devolverá `true` si el entorno coincide con alguno de los valores dados:

    if (App::environment('local')) {
        // The environment is local
    }
    
    if (App::environment(['local', 'staging'])) {
        // The environment is either local OR staging...
    }
    

> {tip} La detección del entorno de la aplicación actual puede ser anulada por una variable de entorno `APP_ENV` a nivel de servidor. Esto puede ser útil cuando necesite compartir la misma aplicación para diferentes configuraciones de entorno, de modo que se puede configurar un host determinado para que coincida con un entorno determinado en las configuraciones de su servidor.

<a name="accessing-configuration-values"></a>

## Acceso a Valores de Configuración

Se puede acceder fácilmente a los valores de configuración utilizando el *helper* global `config` desde cualquier lugar de la aplicación. Los valores de configuración pueden accederse mediante una sintaxis de "punto", que incluye el nombre del archivo y la opción a acceder. Se puede especificar un valor por defecto en caso de que la opción de configuración no exista:

    $value = config('app.timezone');
    

Para establecer valores de configuración en tiempo de ejecución, pasar un *array* al *helper* `config`:

    config(['app.timezone' => 'America/Chicago']);
    

<a name="configuration-caching"></a>

## Configuración de Caché

Para mejorar el rendimiento de la aplicación, es recomendable cachear todos los archivos de configuración en uno solo utilizando el comando de *Artisan* `config:cache`. Esto combinará todas las opciones de configuración de la aplicación en un solo archivo que será cargado rápidamente por el framework.

Normalmente se debe ejecutar el comando `php artisan config:cache` como parte de la rutina de implementación de producción. El comando no debe ejecutarse durante el desarrollo local ya que las opciones cambiarán de forma frecuente durante esta etapa.

> {note} Si ejecuta el comando `config:cache` durante el proceso de despliegue, debe asegurarse de que sólo está llamando a la función `env` desde los archivos de configuración.

<a name="maintenance-mode"></a>

## Modo de Mantenimiento

Cuando la aplicación está en modo de mantenimiento, se mostrará una vista personalizada para todas las solicitudes. Esto hace muy sencillo "desactivar" la aplicación mientras se esta actualizando o efectuando mantenimiento. Se incluye un middleware que controla el modo de mantenimiento de la aplicación. Si la aplicación está en modo de mantenimiento, se lanzará una excepcion `MaintenanceModeException` con un código de estado de 503.

Para activar el modo de mantenimiento, simplemente ejecutar el comando de Artisan `down`:

    php artisan down
    

También se pueden proporcionar las opciones `message` y `retry` para el comando `down`. El valor `message` puede utilizarse para mostrar o registrar un mensaje personalizado, mientras que el valor `retry` se fijará como valor `Retry-After` en la cabecera HTTP:

    php artisan down --message="Upgrading Database" --retry=60
    

Para desactivar el modo de mantenimiento, utilizar el comando `up`:

    php artisan up
    

> {tip} Puede personalizar la plantilla del modo de mantenimiento predeterminado definiendo su propia plantilla en `resources/views/errors/503.blade.php`.

#### Modo de Mantenimiento & Colas

Mientras que la aplicación se encuentra en modo de mantenimiento, no se atenderán [colas de trabajo](/docs/{{version}}/queues). Los trabajos continuarán normalmente una vez que la aplicación salga del modo de mantenimiento.

#### Alternativas al Modo de Mantenimiento

Puesto que el modo de mantenimiento requiere que su aplicación tenga varios segundos de tiempo de inactividad, considere alternativas como [Envoyer](https://envoyer.io) para lograr un despliegue de tiempo de inactividad cero con Laravel.