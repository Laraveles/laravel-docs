# Configuración

- [Introducción](#introduction)
- [Configuración del entorno](#environment-configuration) 
    - [Obtener la configuración del entorno](#retrieving-environment-configuration)
    - [Determinar el entorno actual](#determining-the-current-environment)
- [Acceso a los valores de configuración](#accessing-configuration-values)
- [Configuración de almacenamiento caché](#configuration-caching)
- [Modo mantenimiento](#maintenance-mode)

<a name="introduction"></a>

## Introducción

Todos los archivos de configuración del *framework* Laravel están almacenados en el directorio `config`. Cada opción está documentada, así que no dude en consultar los archivos y familiarizarse con las diferentes opciones disponibles.

<a name="environment-configuration"></a>

## Configuración del entorno

A menudo es útil tener diferentes valores de configuración basados en el entorno donde se ejecute la aplicación. Por ejemplo, se puede utilizar un controlador de *cache* local diferente que el que se utiliza en el servidor de producción.

Para hacer esto más fácil, Laravel utiliza la librería de PHP [DotEnv](https://github.com/vlucas/phpdotenv) de Vance Lucas. En una instalación nueva de Laravel, el directorio raíz de la aplicación contendrá un archivo `.env.example`. Si usted instala Laravel través de *Composer*, este archivo automáticamente será renombrado como `.env`. De lo contrario, usted debe renombrarlo manualmente.

Su fichero `.env` no debería ser incorporado al control de versiones de su aplicación, pues cada desarrollador / servidor que utiliza la aplicación podría requerir una configuración de entorno diferente. Además, esto supondría un riesgo de seguridad en caso de que un intruso acceda a su repositorio de control de versiones, ya que cualquier credencial sensible quedaría expuesta.

Si usted está desarrollando con un equipo, puede continuar incluyendo un archivo `.env.example` con su aplicación. Estableciendo valores de ejemplo en el archivo de configuración de ejemplo, otros desarrolladores en su equipo pueden ver claramente que variables de entorno son necesarias para ejecutar su aplicación. Usted también puede crear un archivo `.env.testing`. Este archivo reemplazará el `.env` al ejecutar los *tests* de PHPUnit o ejecutar los comandos de Artisan con la opción `--env=testing`.

> {tip} Cualquier variable en su archivo `.env` puede ser reemplazada por variables de entorno externas a nivel de servidor o de sistema.

<a name="retrieving-environment-configuration"></a>

### Obtener la configuración del entorno

Todas las variables listadas en este archivo se cargarán en la variable super-global de PHP `$_ENV` cada vez que la aplicación reciba una petición. Sin embargo, usted puede utilizar el *helper* `env` para recuperar valores de esas variables en sus archivos de configuración. De hecho, si revisa los archivos de configuración de Laravel, puede observar que varias de las opciones ya usan este *helper*:

    'debug' => env('APP_DEBUG', false),
    

El segundo valor pasado a la función `env` es el *"valor por defecto"*. Este valor será usado si no existe ninguna variable de entorno para la clave dada.

<a name="determining-the-current-environment"></a>

### Determinando el entorno actual

El entorno actual de la aplicación es determinado a través de la variable `APP_ENV` dede el archivo `.env`. Usted puede acceder a este valor mediante el método `environment` en la [facade](/docs/{{version}}/facades) `App`:

    $environment = App::environment();
    

También puede pasar argumentos al método `environment` para comprobar si hay coincidencia en el entorno para un valor dado. El método retornará `true` si el entorno tiene coincidencia con alguno de los valores dados:

    if (App::environment('local')) {
        // The environment is local
    }
    
    if (App::environment(['local', 'staging'])) {
        // The environment is either local OR staging...
    }
    

> {tip} La detección del entorno de la aplicación actual puede ser anulada por una variable de entorno `APP_ENV` a nivel de servidor. Esto puede ser útil cuando necesite compartir la misma aplicación para diferentes entornos de configuración, de modo que se puede configurar un host para que coincida con un entorno dado en las configuraciones en su servidor.

<a name="accessing-configuration-values"></a>

## Acceso a valores de configuración

Usted puede acceder fácilmente a los valores de configuración utilizando el *helper* global `config` desde cualquier lugar de la aplicación. Los valores de configuración pueden ser accedidos mediante una sintaxis de "punto", que incluye el nombre del archivo y la opción a la que se desea acceder. Se puede especificar un valor por defecto y será devuelto si la opción de configuración no existe:

    $value = config('app.timezone');
    

Para establecer valores de configuración en tiempo de ejecución, pasa una matríz al *helper* `config`:

    config(['app.timezone' => 'America/Chicago']);
    

<a name="configuration-caching"></a>

## Configuración de caché

Para mejorar el rendimiento de su aplicación, debería almacenar en la caché todos los archivos de configuración en un solo archivo utilizando el comando de *Artisan* `config:cache`. Esto combinará todas las opciones de configuración para su aplicación en un solo archivo que será cargado rápidamente por el * framework*.

Usted normalmente debe ejecutar el comando `php artisan config:cache` como parte de la rutina de implementación en producción. El comando no se debe ejecutarse durante el desarrollo local ya que las opciones de configuración necesitarán ser cambiadas frecuentemente durante el desarrollo de la aplicación.

> {note} Si usted ejecuta el comando `config:cache` durante su proceso de despliegue, debe asegurarse que sólo está llamando a la función `env` desde los archivos de configuración.

<a name="maintenance-mode"></a>

## Modo mantenimiento

Cuando su aplicación está en modo de mantenimiento, una vista personalizada será mostrada para todas las peticiones en su aplicación. Esto hace muy sencillo "desactivar" la aplicación mientras se está actualizando o cuando se está realizando el mantenimiento. La comprobación del modo de mantenimiento está en la pila, por defecto, de *middleware* de su aplicación. Si la aplicación está en modo de mantenimiento, una excepción `MaintenanceModeException` será lanzada con un código de estado 503.

Para activar el modo de mantenimiento, simplemente ejecutar el comando de Artisan `down`:

    php artisan down
    

También se pueden proporcionar las opciones `message` y `retry` para el comando `down`. El valor `message` puede ser utilizado para mostrar o registrar un mensaje personalizado, mientras que el valor `retry` será establecido como valor `Retry-After` en la cabecera HTTP:

    php artisan down --message="Upgrading Database" --retry=60
    

Para desactivar el modo de mantenimiento, utilice el comando `up`:

    php artisan up
    

> {tip} Puede personalizar la plantilla del modo de mantenimiento predeterminado definiendo su propia plantilla en `resources/views/errors/503.blade.php`.

#### Modo mantenimiento & colas

Mientras que la aplicación se encuentra en modo de mantenimiento, no se atenderán [trabajos encolados](/docs/{{version}}/queues). Los trabajos continuarán siendo manejados normalmente una vez que la aplicación salga del modo de mantenimiento.

#### Alternativas al modo de mantenimiento

Puesto que el modo de mantenimiento requiere que su aplicación tenga varios segundos de tiempo de inactividad, considere alternativas como [Envoyer](https://envoyer.io) para lograr un despliegue con tiempo de inactividad cero con Laravel.