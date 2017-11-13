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

The `routes` directory contains all of the route definitions for your application. By default, several route files are included with Laravel: `web.php`, `api.php`, `console.php` and `channels.php`.

The `web.php` file contains routes that the `RouteServiceProvider` places in the `web` middleware group, which provides session state, CSRF protection, and cookie encryption. If your application does not offer a stateless, RESTful API, all of your routes will most likely be defined in the `web.php` file.

The `api.php` file contains routes that the `RouteServiceProvider` places in the `api` middleware group, which provides rate limiting. These routes are intended to be stateless, so requests entering the application through these routes are intended to be authenticated via tokens and will not have access to session state.

The `console.php` file is where you may define all of your Closure based console commands. Each Closure is bound to a command instance allowing a simple approach to interacting with each command's IO methods. Even though this file does not define HTTP routes, it defines console based entry points (routes) into your application.

The `channels.php` file is where you may register all of the event broadcasting channels that your application supports.

<a name="the-storage-directory"></a>

#### The Storage Directory

The `storage` directory contains your compiled Blade templates, file based sessions, file caches, and other files generated by the framework. This directory is segregated into `app`, `framework`, and `logs` directories. The `app` directory may be used to store any files generated by your application. The `framework` directory is used to store framework generated files and caches. Finally, the `logs` directory contains your application's log files.

The `storage/app/public` directory may be used to store user-generated files, such as profile avatars, that should be publicly accessible. You should create a symbolic link at `public/storage` which points to this directory. You may create the link using the `php artisan storage:link` command.

<a name="the-tests-directory"></a>

#### The Tests Directory

The `tests` directory contains your automated tests. An example [PHPUnit](https://phpunit.de/) is provided out of the box. Each test class should be suffixed with the word `Test`. You may run your tests using the `phpunit` or `php vendor/bin/phpunit` commands.

<a name="the-vendor-directory"></a>

#### The Vendor Directory

The `vendor` directory contains your [Composer](https://getcomposer.org) dependencies.

<a name="the-app-directory"></a>

## The App Directory

The majority of your application is housed in the `app` directory. By default, this directory is namespaced under `App` and is autoloaded by Composer using the [PSR-4 autoloading standard](http://www.php-fig.org/psr/psr-4/).

The `app` directory contains a variety of additional directories such as `Console`, `Http`, and `Providers`. Think of the `Console` and `Http` directories as providing an API into the core of your application. The HTTP protocol and CLI are both mechanisms to interact with your application, but do not actually contain application logic. In other words, they are simply two ways of issuing commands to your application. The `Console` directory contains all of your Artisan commands, while the `Http` directory contains your controllers, middleware, and requests.

A variety of other directories will be generated inside the `app` directory as you use the `make` Artisan commands to generate classes. So, for example, the `app/Jobs` directory will not exist until you execute the `make:job` Artisan command to generate a job class.

> {tip} Many of the classes in the `app` directory can be generated by Artisan via commands. To review the available commands, run the `php artisan list make` command in your terminal.

<a name="the-console-directory"></a>

#### The Console Directory

The `Console` directory contains all of the custom Artisan commands for your application. These commands may be generated using the `make:command` command. This directory also houses your console kernel, which is where your custom Artisan commands are registered and your [scheduled tasks](/docs/{{version}}/scheduling) are defined.

<a name="the-events-directory"></a>

#### The Events Directory

This directory does not exist by default, but will be created for you by the `event:generate` and `make:event` Artisan commands. The `Events` directory, as you might expect, houses [event classes](/docs/{{version}}/events). Events may be used to alert other parts of your application that a given action has occurred, providing a great deal of flexibility and decoupling.

<a name="the-exceptions-directory"></a>

#### The Exceptions Directory

The `Exceptions` directory contains your application's exception handler and is also a good place to place any exceptions thrown by your application. If you would like to customize how your exceptions are logged or rendered, you should modify the `Handler` class in this directory.

<a name="the-http-directory"></a>

#### The Http Directory

The `Http` directory contains your controllers, middleware, and form requests. Almost all of the logic to handle requests entering your application will be placed in this directory.

<a name="the-jobs-directory"></a>

#### The Jobs Directory

This directory does not exist by default, but will be created for you if you execute the `make:job` Artisan command. The `Jobs` directory houses the [queueable jobs](/docs/{{version}}/queues) for your application. Jobs may be queued by your application or run synchronously within the current request lifecycle. Jobs that run synchronously during the current request are sometimes referred to as "commands" since they are an implementation of the [command pattern](https://en.wikipedia.org/wiki/Command_pattern).

<a name="the-listeners-directory"></a>

#### The Listeners Directory

This directory does not exist by default, but will be created for you if you execute the `event:generate` or `make:listener` Artisan commands. The `Listeners` directory contains the classes that handle your [events](/docs/{{version}}/events). Event listeners receive an event instance and perform logic in response to the event being fired. For example, a `UserRegistered` event might be handled by a `SendWelcomeEmail` listener.

<a name="the-mail-directory"></a>

#### The Mail Directory

This directory does not exist by default, but will be created for you if you execute the `make:mail` Artisan command. The `Mail` directory contains all of your classes that represent emails sent by your application. Mail objects allow you to encapsulate all of the logic of building an email in a single, simple class that may be sent using the `Mail::send` method.

<a name="the-notifications-directory"></a>

#### The Notifications Directory

This directory does not exist by default, but will be created for you if you execute the `make:notification` Artisan command. The `Notifications` directory contains all of the "transactional" notifications that are sent by your application, such as simple notifications about events that happen within your application. Laravel's notification features abstracts sending notifications over a variety of drivers such as email, Slack, SMS, or stored in a database.

<a name="the-policies-directory"></a>

#### The Policies Directory

This directory does not exist by default, but will be created for you if you execute the `make:policy` Artisan command. The `Policies` directory contains the authorization policy classes for your application. Policies are used to determine if a user can perform a given action against a resource. For more information, check out the [authorization documentation](/docs/{{version}}/authorization).

<a name="the-providers-directory"></a>

#### The Providers Directory

The `Providers` directory contains all of the [service providers](/docs/{{version}}/providers) for your application. Service providers bootstrap your application by binding services in the service container, registering events, or performing any other tasks to prepare your application for incoming requests.

In a fresh Laravel application, this directory will already contain several providers. You are free to add your own providers to this directory as needed.

<a name="the-rules-directory"></a>

#### The Rules Directory

This directory does not exist by default, but will be created for you if you execute the `make:rule` Artisan command. The `Rules` directory contains the custom validation rule objects for your application. Rules are used to encapsulate complicated validation logic in a simple object. For more information, check out the [validation documentation](/docs/{{version}}/validation).