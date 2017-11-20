# Estructura de directorios

- [Introducción](#introduction)
- [El directorio *raíz*](#the-root-directory) 
    - [El directorio `app`](#the-root-app-directory)
    - [El directorio `bootstrap`](#the-bootstrap-directory)
    - [El directorio `config`](#the-config-directory)
    - [El directorio `database`](#the-database-directory)
    - [El directorio `public`](#the-public-directory)
    - [El directorio `resources`](#the-resources-directory)
    - [El directorio `routes`](#the-routes-directory)
    - [El directorio `storage`](#the-storage-directory)
    - [El directorio `tests`](#the-tests-directory)
    - [El directorio `vendor`](#the-vendor-directory)
- [El directorio *app*](#the-app-directory) 
    - [El directorio `Console`](#the-console-directory)
    - [El directorio `Events`](#the-events-directory)
    - [El directorio `Exceptions`](#the-exceptions-directory)
    - [El directorio `Http`](#the-http-directory)
    - [El directorio `Jobs`](#the-jobs-directory)
    - [El directorio `Listeners`](#the-listeners-directory)
    - [El directorio `Mail`](#the-mail-directory)
    - [El directorio `Notifications`](#the-notifications-directory)
    - [El directorio `Policies`](#the-policies-directory)
    - [El directorio `Providers`](#the-providers-directory)
    - [El directorio `Rules`](#the-rules-directory)

<a name="introduction"></a>

## Introducción

La estructura predeterminada de Laravel pretende proporcionar un punto de partida ideal tanto para grandes como para pequeñas aplicaciones. Por supuesto, se puede organizar la aplicación a placer. Laravel apenas impone restricciones sobre donde almacenar una clase - siempre y cuando *Composer* pueda cargarla.

#### ¿Dónde está el directorio para modelos?

Cuando se empieza con Laravel, muchos programadores se confunden al no encontrar una carpeta específica para `modelos`. Sin embargo, la falta de este directorio es intencional. La palabra *models* es ambigua puesto que puede significar cosas diferentes para gente diferente. Algunos programadores se refieren a "modelo" como la totalidad de la lógica de negocio de su aplicación, mientras que otros solo a aquella que interactúa con una base de datos relacional.

Es por esta razón que se establecen los modelos de Eloquent en el directorio `app` por defecto, permitiendo al programador guardarlos en cualquier otro sitio que elija.

<a name="the-root-directory"></a>

## El directorio *raíz*

<a name="the-root-app-directory"></a>

#### El directorio *app*

La carpeta `app`, tal y como se puede esperar, contiene el núcleo del código de la aplicación. Se explorará este directorio en detalle a continuación; casi todas las clases de una aplicación estarán dentro de esta carpeta.

<a name="the-bootstrap-directory"></a>

#### El directorio *bootstrap*

El directorio `bootstrap` contiene el archivo `app.php` el cual envuelve el framework. Esta carpeta contiene además el directorio `cache`, la cual contiene los archivos generados por el framework para mejorar su rendimiento como los archivos de caché de rutas y servicios.

<a name="the-config-directory"></a>

#### El directorio *config*

El directorio `config`, como su nombre indica, contiene todos los archivos de configuración de la aplicación. Es una buena idea revisar el contenido de todos estos archivos para familiarizarse con las opciones disponibles.

<a name="the-database-directory"></a>

#### El directorio *database*

La carpeta `database` contiene las migraciones de la base de datos y semillas. Si se desea, se puede utilizar esta carpeta para mantener una base de datos SQLite.

<a name="the-public-directory"></a>

#### El directorio *public*

El directorio `public` contiene el archivo `index.php`, el cual es el punto de acceso para todas las peticiones que accedan a la aplicación y configura el *autoloading*. Este directorio contendrá también los recursos JavaScript, CSS, imágenes y otros.

<a name="the-resources-directory"></a>

#### El directorio *resources*

El directorio `resources` contiene todas las vistas, así como todos los recursos LESS, SASS, JavaSscript y otros archivos sin compilar. Este directorio también incluye los archivos de idioma.

<a name="the-routes-directory"></a>

#### El directorio *routes*

El directorio `routes` contiene todas las definiciones de rutas de la aplicación. Por defecto, varios archivos de rutas se incluyen con Laravel: `web.php` `api.php`, `console.php` y `channels.php`.

El archivo `web.php` contiene rutas que el `RouteServiceProvider` coloca en el grupo de * middleware * `web`, el cual provee el estado de la sesión, protección CSRF y cifrado de cookies. Si la aplicación no ofrecera una * API RESTful * que sea *Stateless*, lo mas probable es que todas las rutas sean definidas en el archivo `web.php`.

El archivo `api.php` contiene rutas que el `RouteServiceProvider` coloca en el grupo de *middleware* `api`, que proporciona limites de velocidad. Estas rutas estan destinadas a ser * stateless *, por lo cual los * requests * que accesan a la aplicación mediante dichas rutas están destinados a ser autenticados mediante * tokens * y no tendrán acceso al estado de sesión.

El archivo `console.php` es donde se definen todos los comandos de consola basados en * Closures *. Cada * Closure * esta ligada a una instancia de comando permitiendo un enfoque simple para interactuar con cada uno de los métodos * IO * de los comandos. Aunque este archivo no define rutas HTTP, define puntos de entrada(rutas) basados en la consola en la aplicación.

El archivo `channels.php` es donde se deben registrar todos los canales que emitan eventos que sean compatibles con la aplicación.

<a name="the-storage-directory"></a>

#### El directorio *storage*

El directorio `storage` contiene todas plantillas Blade compiladas, sesiones y caches basados en archivos y otros archivos generados por el framework. Esta carpeta se encuentra separada en las carpetas `app`, `framework` y `logs`. La carpeta `app` debe ser usada para almacenar cualquier archivo generado por la aplicación. El directorio `framework` se utiliza para almacenar archivos generados por el framework y caches. Finalmente, el directorio de `log` contiene los registros de logging de la aplicación.

El directorio `storage/app/public` puede ser usado para almacenar archivos generados por el usuario, tales como fotos de perfil, que deberían ser accesibles publicamente. Se debería crear un enlace simbólico en `public/storage` que apunte a esta carpeta. Se puede crear el enlace utilizando el comando `php artisan storage:link`.

<a name="the-tests-directory"></a>

#### El directorio *tests*

El directorio `tests` contiene las pruebas automatizadas. Se proporciona el ejemplo [PHPUnit](https://phpunit.de/) de fabrica. Cada clase test debería poseer el sufijo `Test`. Se pueden ejecutar las pruebas utilizando los comandos `phpunit` o `php vendor/bin/phpunit`.

<a name="the-vendor-directory"></a>

#### El directorio *vendor*

El directorio `vendor` contiene las dependencias de [Composer](https://getcomposer.org).

<a name="the-app-directory"></a>

## El directorio *app*

La mayor parte de la aplicación reside en el directorio `app`. Por defecto, este directorio contiene el namespace `App` y se carga con Composer utilizando el [estándar de carga automática PSR-4](http://www.php-fig.org/psr/psr-4/).

El directorio `app` contiene una variedad de directorios adicionales tales como `Console`, `Http` y `Providers`. Los directorios `Console` y `Http` proveen un API al núcleo de la aplicación. Tanto el protocolo HTTP como CLI son mecanismos para interactuar con la aplicación, pero no contienen lógica. En otras palabras, son simplemente dos formas de emitir órdenes a la aplicación. El directorio `Console` contiene todos los comandos de Artisan, mientras que `Http` contiene los controladores, filtros y *requests*.

Al utilizar los comandos `make` de Artisan para crear clases se crearan distintas carpetas dentro del directorio `app`. Por ejemplo la carpeta `app/Jobs` no existirá hasta que se ejecute el comando `make:job` para crear un job.

> **{tip}** Muchas de las clases en el directorio `app` pueden ser crear mediante comandos Artisan. Para listar los comandos disponibles, ejecutar el siguiente comando en la terminal `php artisan list make`.

<a name="the-console-directory"></a>

#### El directorio *Console*

The `Console` directory contains all of the custom Artisan commands for your application. These commands may be generated using the `make:command` command. This directory also houses your console kernel, which is where your custom Artisan commands are registered and your [scheduled tasks](/docs/{{version}}/scheduling) are defined.

<a name="the-events-directory"></a>

#### El directorio *Events*

This directory does not exist by default, but will be created for you by the `event:generate` and `make:event` Artisan commands. El directorio `Events`, como era de esperar, almacena [clases de eventos](/docs/{{version}}/events). Los eventos se utilizan para alertar a otras partes de la aplicación de que una acción concreta ha ocurrido, proporcionando flexibilidad y desacoplamiento.

<a name="the-exceptions-directory"></a>

#### El directorio *Exceptions*

The `Exceptions` directory contains your application's exception handler and is also a good place to place any exceptions thrown by your application. If you would like to customize how your exceptions are logged or rendered, you should modify the `Handler` class in this directory.

<a name="the-http-directory"></a>

#### El directorio *Http*

The `Http` directory contains your controllers, middleware, and form requests. Almost all of the logic to handle requests entering your application will be placed in this directory.

<a name="the-jobs-directory"></a>

#### El directorio *Jobs*

This directory does not exist by default, but will be created for you if you execute the `make:job` Artisan command. The `Jobs` directory houses the [queueable jobs](/docs/{{version}}/queues) for your application. Jobs may be queued by your application or run synchronously within the current request lifecycle. Jobs that run synchronously during the current request are sometimes referred to as "commands" since they are an implementation of the [command pattern](https://en.wikipedia.org/wiki/Command_pattern).

<a name="the-listeners-directory"></a>

#### El directorio *Listeners*

This directory does not exist by default, but will be created for you if you execute the `event:generate` or `make:listener` Artisan commands. The `Listeners` directory contains the classes that handle your [events](/docs/{{version}}/events). Event listeners receive an event instance and perform logic in response to the event being fired. Por ejemplo, un evento `UserRegistered` puede gestionarse por un listener `SendWelcomeEmail`.

<a name="the-mail-directory"></a>

#### El directorio *Mail*

This directory does not exist by default, but will be created for you if you execute the `make:mail` Artisan command. The `Mail` directory contains all of your classes that represent emails sent by your application. Mail objects allow you to encapsulate all of the logic of building an email in a single, simple class that may be sent using the `Mail::send` method.

<a name="the-notifications-directory"></a>

#### El directorio *Notifications*

This directory does not exist by default, but will be created for you if you execute the `make:notification` Artisan command. The `Notifications` directory contains all of the "transactional" notifications that are sent by your application, such as simple notifications about events that happen within your application. Laravel's notification features abstracts sending notifications over a variety of drivers such as email, Slack, SMS, or stored in a database.

<a name="the-policies-directory"></a>

#### El directorio *Policies*

This directory does not exist by default, but will be created for you if you execute the `make:policy` Artisan command. The `Policies` directory contains the authorization policy classes for your application. Policies are used to determine if a user can perform a given action against a resource. For more information, check out the [authorization documentation](/docs/{{version}}/authorization).

<a name="the-providers-directory"></a>

#### El directorio *Providers*

The `Providers` directory contains all of the [service providers](/docs/{{version}}/providers) for your application. Service providers bootstrap your application by binding services in the service container, registering events, or performing any other tasks to prepare your application for incoming requests.

In a fresh Laravel application, this directory will already contain several providers. You are free to add your own providers to this directory as needed.

<a name="the-rules-directory"></a>

#### El directorio *Rules*

This directory does not exist by default, but will be created for you if you execute the `make:rule` Artisan command. The `Rules` directory contains the custom validation rule objects for your application. Rules are used to encapsulate complicated validation logic in a simple object. For more information, check out the [validation documentation](/docs/{{version}}/validation).