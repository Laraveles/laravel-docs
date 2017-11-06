# Service Providers

- [Introducción](#introduction)
- [Escribiendo *Service Providers*](#writing-service-providers) 
    - [El método *Register*](#the-register-method)
    - [El método *Boot*](#the-boot-method)
- [Registrando *Providers*](#registering-providers)
- [Providers diferidos](#deferred-providers)

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

Como se ha comentado anteriormente, dentro del método `register`, únicamente se deben enlazar cosas dentro del [service container](/docs/{{version}}/container). Nunca debe intentar registrar ningún escuchador de eventos, rutas, o cualquier otra funcionalidad dentro del método `register`. Por otra parte, en ocasiones se puede usar un servicio proporcionado por un *service provider* que no haya sido cargado todavía.

A continuación se muestra un ejemplo de un *service provider* básico. Dentro de cualquiera de los métodos de un *service provider*, siempre está disponible a la propiedad `$app` que proporciona acceso al *service container*:

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
    

Este *service provider* únicamente define el método `register` y lo utiliza para definir una implementación de `Riak\Connection` en el *service container*. Si no se entiende cómo funciona el *service container*, se puede consultar su [documentación](/docs/{{version}}/container).

<a name="the-boot-method"></a>

### El método *Boot*

Entonces, ¿qué ocurre si hay que registrar un *view composer* en un *service provider*? Suele hacerse dentro del método `boot`. **A este método se le llama después de que todos los demás *service providers* han sido registrados**, por tanto se tiene acceso a todos los demás servicios que hayan sido registrados por el framework:

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
    

#### El Método *Boot*, inyección de dependencias

Se pueden añadir dependencias para el método `boot` del *service provider*. El [service container](/docs/{{version}}/container) inyectará automáticamente cualquier dependencia que se necesite:

    use Illuminate\Contracts\Routing\ResponseFactory;
    
    public function boot(ResponseFactory $response)
    {
        $response->macro('caps', function ($value) {
            //
        });
    }
    

<a name="registering-providers"></a>

## Registrando *Providers*

Todos los service providers están registrados en el archivo de configuración `config/app.php`. Este archivo contiene un *array* `providers` donde se pueden enumerar los nombres de los *service providers*. Por defecto se incluye un conjunto de *service providers* del núcleo de Laravel. Estos *providers* inicializan los componentes base de Laravel como el *mailer*, *queue*, *cache* y otros.

Para registrar un providers, simplemente hay que añadirlo al array:

    'providers' => [
        // Other Service Providers
    
        App\Providers\ComposerServiceProvider::class,
    ],
    

<a name="deferred-providers"></a>

## Providers diferidos

Si el proveedor **sólo** registra enlaces en el [service container](/docs/{{version}}/container), se puede diferir este registro hasta que uno de estos enlaces se necesite realmente. Aplazando la carga de estos providers se incrementará el rendimiento de la aplicación, puesto que no se cargará desde el sistema de archivo en cada petición.

Laravel complia y almacena una lista de todos los services proporcionados por los service providers diferidos junto con el nombre de su clase. Entonces, sólo cuando se intente resolver uno de estos servicios Laravel cargará el service provider.

Para aplazar la carga de un provider, establecer la propiedad `defer` a `true` y definir el método `provides`. El método `provides` debe retornar los *bindings* del *service container* registrados:

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