# Desarrollo de paquetes

- [Introducción](#introduction) 
    - [Nota sobre *facades*](#a-note-on-facades)
- [Descubrir paquetes](#package-discovery)
- [*Service Providers*](#service-providers)
- [Recursos – *Resources*](#resources) 
    - [Configuración](#configuration)
    - [Migraciones](#migrations)
    - [Rutas](#routes)
    - [Traducciones](#translations)
    - [Vistas](#views)
- [Comandos](#commands)
- [Publicar *assets*](#public-assets)
- [Publicar grupos de ficheros](#publishing-file-groups)

<a name="introduction"></a>

## Introducción

Los paquetes son el método principal para añadir funcionalidad a Laravel. Pueden ser cualquier cosa, desde una gran manera de trabajar con fechas como [Carbon](https://github.com/briannesbitt/Carbon), o un *framework* BDD completo como [Behat](https://github.com/Behat/Behat).

Por supuesto, hay diferentes tipos de paquetes. Algunos paquetes son independientes, lo que significa que funcionan con cualquier framework PHP. Carbon y Behat son ejemplos de paquetes independientes. Cualquiera de estos paquetes puede utilizarse en Laravel con tan solo añadirlos en el archivo `composer.json`.

Por otro lado, otros paquetes son específicos, a propósito, para utilizarlos con Laravel. Estos paquetes pueden tener rutas, controladores, vistas y configuración específicamente desarrollada para una aplicación Laravel. Esta guía principalmente cubre el desarrollo de esos paquetes específicos para Laravel.

<a name="a-note-on-facades"></a>

### Nota sobre *facades*

Al escribir una aplicación de Laravel, generalmente no importa si usa contratos o *facades* ya que ambos proporcionan niveles esencialmente iguales de *testing*. Sin embargo, su paquete no tendrá acceso a todos los *helpers* de *testing* de Laravel. Si desea poder escribir sus *tests* como si existieran dentro de una aplicación típica de Laravel, puede utilizar el paquete [Orchestral Testbench](https://github.com/orchestral/testbench).

<a name="package-discovery"></a>

## Descubrir paquetes

En el fichero de configuración `config/app.php` de Laravel, la opción `providers` define una lista de *service providers* que Laravel debe cargar. When someone installs your package, you will typically want your service provider to be included in this list. Instead of requiring users to manually add your service provider to the list, you may define the provider in the `extra` section of your package's `composer.json` file. In addition to service providers, you may also list any [facades](/docs/{{version}}/facades) you would like to be registered:

    "extra": {
        "laravel": {
            "providers": [
                "Barryvdh\\Debugbar\\ServiceProvider"
            ],
            "aliases": {
                "Debugbar": "Barryvdh\\Debugbar\\Facade"
            }
        }
    },
    

Once your package has been configured for discovery, Laravel will automatically register its service providers and facades when it is installed, creating a convenient installation experience for your package's users.

### Opting Out Of Package Discovery

If you are the consumer of a package and would like to disable package discovery for a package, you may list the package name in the `extra` section of your application's `composer.json` file:

    "extra": {
        "laravel": {
            "dont-discover": [
                "barryvdh/laravel-debugbar"
            ]
        }
    },
    

You may disable package discovery for all packages using the `*` character inside of your application's `dont-discover` directive:

    "extra": {
        "laravel": {
            "dont-discover": [
                "*"
            ]
        }
    },
    

<a name="service-providers"></a>

## Service Providers

[Service providers](/docs/{{version}}/providers) are the connection points between your package and Laravel. A service provider is responsible for binding things into Laravel's [service container](/docs/{{version}}/container) and informing Laravel where to load package resources such as views, configuration, and localization files.

A service provider extends the `Illuminate\Support\ServiceProvider` class and contains two methods: `register` and `boot`. The base `ServiceProvider` class is located in the `illuminate/support` Composer package, which you should add to your own package's dependencies. To learn more about the structure and purpose of service providers, check out [their documentation](/docs/{{version}}/providers).

<a name="resources"></a>

## Resources

<a name="configuration"></a>

### Configuration

Typically, you will need to publish your package's configuration file to the application's own `config` directory. This will allow users of your package to easily override your default configuration options. To allow your configuration files to be published, call the `publishes` method from the `boot` method of your service provider:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/path/to/config/courier.php' => config_path('courier.php'),
        ]);
    }
    

Now, when users of your package execute Laravel's `vendor:publish` command, your file will be copied to the specified publish location. Of course, once your configuration has been published, its values may be accessed like any other configuration file:

    $value = config('courier.option');
    

> {note} You should not define Closures in your configuration files. They can not be serialized correctly when users execute the `config:cache` Artisan command.

#### Default Package Configuration

You may also merge your own package configuration file with the application's published copy. This will allow your users to define only the options they actually want to override in the published copy of the configuration. To merge the configurations, use the `mergeConfigFrom` method within your service provider's `register` method:

    /**
     * Register bindings in the container.
     *
     * @return void
     */
    public function register()
    {
        $this->mergeConfigFrom(
            __DIR__.'/path/to/config/courier.php', 'courier'
        );
    }
    

> {note} This method only merges the first level of the configuration array. If your users partially define a multi-dimensional configuration array, the missing options will not be merged.

<a name="routes"></a>

### Routes

If your package contains routes, you may load them using the `loadRoutesFrom` method. This method will automatically determine if the application's routes are cached and will not load your routes file if the routes have already been cached:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadRoutesFrom(__DIR__.'/routes.php');
    }
    

<a name="migrations"></a>

### Migrations

If your package contains [database migrations](/docs/{{version}}/migrations), you may use the `loadMigrationsFrom` method to inform Laravel how to load them. The `loadMigrationsFrom` method accepts the path to your package's migrations as its only argument:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadMigrationsFrom(__DIR__.'/path/to/migrations');
    }
    

Once your package's migrations have been registered, they will automatically be run when the `php artisan migrate` command is executed. You do not need to export them to the application's main `database/migrations` directory.

<a name="translations"></a>

### Translations

If your package contains [translation files](/docs/{{version}}/localization), you may use the `loadTranslationsFrom` method to inform Laravel how to load them. For example, if your package is named `courier`, you should add the following to your service provider's `boot` method:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
    }
    

Package translations are referenced using the `package::file.line` syntax convention. So, you may load the `courier` package's `welcome` line from the `messages` file like so:

    echo trans('courier::messages.welcome');
    

#### Publishing Translations

If you would like to publish your package's translations to the application's `resources/lang/vendor` directory, you may use the service provider's `publishes` method. The `publishes` method accepts an array of package paths and their desired publish locations. For example, to publish the translation files for the `courier` package, you may do the following:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
    
        $this->publishes([
            __DIR__.'/path/to/translations' => resource_path('lang/vendor/courier'),
        ]);
    }
    

Now, when users of your package execute Laravel's `vendor:publish` Artisan command, your package's translations will be published to the specified publish location.

<a name="views"></a>

### Views

To register your package's [views](/docs/{{version}}/views) with Laravel, you need to tell Laravel where the views are located. You may do this using the service provider's `loadViewsFrom` method. The `loadViewsFrom` method accepts two arguments: the path to your view templates and your package's name. For example, if your package's name is `courier`, you would add the following to your service provider's `boot` method:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
    }
    

Package views are referenced using the `package::view` syntax convention. So, once your view path is registered in a service provider, you may load the `admin` view from the `courier` package like so:

    Route::get('admin', function () {
        return view('courier::admin');
    });
    

#### Overriding Package Views

When you use the `loadViewsFrom` method, Laravel actually registers two locations for your views: the application's `resources/views/vendor` directory and the directory you specify. So, using the `courier` example, Laravel will first check if a custom version of the view has been provided by the developer in `resources/views/vendor/courier`. Then, if the view has not been customized, Laravel will search the package view directory you specified in your call to `loadViewsFrom`. This makes it easy for package users to customize / override your package's views.

#### Publishing Views

If you would like to make your views available for publishing to the application's `resources/views/vendor` directory, you may use the service provider's `publishes` method. The `publishes` method accepts an array of package view paths and their desired publish locations:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
    
        $this->publishes([
            __DIR__.'/path/to/views' => resource_path('views/vendor/courier'),
        ]);
    }
    

Now, when users of your package execute Laravel's `vendor:publish` Artisan command, your package's views will be copied to the specified publish location.

<a name="commands"></a>

## Commands

To register your package's Artisan commands with Laravel, you may use the `commands` method. This method expects an array of command class names. Once the commands have been registered, you may execute them using the [Artisan CLI](/docs/{{version}}/artisan):

    /**
     * Bootstrap the application services.
     *
     * @return void
     */
    public function boot()
    {
        if ($this->app->runningInConsole()) {
            $this->commands([
                FooCommand::class,
                BarCommand::class,
            ]);
        }
    }
    

<a name="public-assets"></a>

## Public Assets

Your package may have assets such as JavaScript, CSS, and images. To publish these assets to the application's `public` directory, use the service provider's `publishes` method. In this example, we will also add a `public` asset group tag, which may be used to publish groups of related assets:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/path/to/assets' => public_path('vendor/courier'),
        ], 'public');
    }
    

Now, when your package's users execute the `vendor:publish` command, your assets will be copied to the specified publish location. Since you will typically need to overwrite the assets every time the package is updated, you may use the `--force` flag:

    php artisan vendor:publish --tag=public --force
    

<a name="publishing-file-groups"></a>

## Publishing File Groups

You may want to publish groups of package assets and resources separately. For instance, you might want to allow your users to publish your package's configuration files without being forced to publish your package's assets. You may do this by "tagging" them when calling the `publishes` method from a package's service provider. For example, let's use tags to define two publish groups in the `boot` method of a package service provider:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/../config/package.php' => config_path('package.php')
        ], 'config');
    
        $this->publishes([
            __DIR__.'/../database/migrations/' => database_path('migrations')
        ], 'migrations');
    }
    

Now your users may publish these groups separately by referencing their tag when executing the `vendor:publish` command:

    php artisan vendor:publish --tag=config