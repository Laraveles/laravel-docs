# Service Providers

- [Introducción](#introduction)
- [Escribiendo *Service Providers*](#writing-service-providers) 
    - [El método *Register*](#the-register-method)
    - [El método *Boot*](#the-boot-method)
- [Registrando *Providers*](#registering-providers)
- [Deferred Providers](#deferred-providers)

<a name="introduction"></a>

## Introducción

Los *service providers* son la base de la inicialización de toda aplicación Laravel. Cualquier aplicación, así como los propios servicios de Laravel, se arrancan a través de *service providers*.

Pero, ¿qué quiere decir "inicializar/arrancar"? En general, se refiere a **registrar** cosas, incluyendo el registro de *bindings* (o enlaces) en el *service container*, *event listeners*, *middleware* e incluso rutas. Los service providers son el lugar central donde configurar la aplicación.

Si se abre el archivo `config/app.php` incluido con Laravel, se podrá observar el *array* `providers`. Son todas las clases de *service providers* que serán cargadas por la aplicación. Por supuesto, muchos de estos de estos *providers* son "diferidos", por lo que no se cargarán en cada petición, sino cuando los servicios que proporcionan sean realmente necesarios.

En esta visión general se aprenderá cómo escribir *service providers* propios y registrarlos en una aplicación Laravel.

<a name="writing-service-providers"></a>

## Escribiendo *Service Providers*

Todos los *service providers* heredan de la clase `Illuminate\Support\ServiceProvider`. La mayoría de ellos contienen un método `register` y otro `boot`. En el método `register`, únicamente se deben **añadir *bindings* (enlazar cosas) al [service container](/docs/{{version}}/container)**. Nunca debe intentar registrar ningún *event listeners*, rutas, o cualquier otra funcionalidad dentro del método `register`.

Se puede generar un nuevo *provider* a través del *Artisan CLI* con el comando `make:provider`:

    php artisan make:provider RiakServiceProvider
    

<a name="the-register-method"></a>

### El método *Register*

Como se ha comentado anteriormente, dentro del método `register`, únicamente se deben enlazar cosas dentro del [service container](/docs/{{version}}/container). Nunca debe intentar registrar ningún escuchador de eventos, rutas, o cualquier otra funcionalidad dentro del método `register`. Otherwise, you may accidentally use a service that is provided by a service provider which has not loaded yet.

Let's take a look at a basic service provider. Within any of your service provider methods, you always have access to the `$app` property which provides access to the service container:

    <?php
    
    namespace App\Providers;
    
    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;
    
    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection(config('riak'));
            });
        }
    }
    

This service provider only defines a `register` method, and uses that method to define an implementation of `Riak\Connection` in the service container. If you don't understand how the service container works, check out [its documentation](/docs/{{version}}/container).

<a name="the-boot-method"></a>

### The Boot Method

So, what if we need to register a view composer within our service provider? This should be done within the `boot` method. **This method is called after all other service providers have been registered**, meaning you have access to all other services that have been registered by the framework:

    <?php
    
    namespace App\Providers;
    
    use Illuminate\Support\ServiceProvider;
    
    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            view()->composer('view', function () {
                //
            });
        }
    }
    

#### Boot Method Dependency Injection

You may type-hint dependencies for your service provider's `boot` method. The [service container](/docs/{{version}}/container) will automatically inject any dependencies you need:

    use Illuminate\Contracts\Routing\ResponseFactory;
    
    public function boot(ResponseFactory $response)
    {
        $response->macro('caps', function ($value) {
            //
        });
    }
    

<a name="registering-providers"></a>

## Registering Providers

All service providers are registered in the `config/app.php` configuration file. This file contains a `providers` array where you can list the class names of your service providers. By default, a set of Laravel core service providers are listed in this array. These providers bootstrap the core Laravel components, such as the mailer, queue, cache, and others.

To register your provider, simply add it to the array:

    'providers' => [
        // Other Service Providers
    
        App\Providers\ComposerServiceProvider::class,
    ],
    

<a name="deferred-providers"></a>

## Deferred Providers

If your provider is **only** registering bindings in the [service container](/docs/{{version}}/container), you may choose to defer its registration until one of the registered bindings is actually needed. Deferring the loading of such a provider will improve the performance of your application, since it is not loaded from the filesystem on every request.

Laravel compiles and stores a list of all of the services supplied by deferred service providers, along with the name of its service provider class. Then, only when you attempt to resolve one of these services does Laravel load the service provider.

To defer the loading of a provider, set the `defer` property to `true` and define a `provides` method. The `provides` method should return the service container bindings registered by the provider:

    <?php
    
    namespace App\Providers;
    
    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;
    
    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Indicates if loading of the provider is deferred.
         *
         * @var bool
         */
        protected $defer = true;
    
        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection($app['config']['riak']);
            });
        }
    
        /**
         * Get the services provided by the provider.
         *
         * @return array
         */
        public function provides()
        {
            return [Connection::class];
        }
    
    }