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

En el fichero de configuración `config/app.php` de Laravel, la opción `providers` define una lista de *service providers* que Laravel debe cargar. Cuando alguien instala su paquete, normalmente querrá que su *service provider* se incluya en esta lista. En lugar de exigir a los usuarios que agreguen manualmente su *service provider* a la lista, puede definirlo en la sección `extra` del archivo `composer.json` de su paquete. Además de los *service provider*, también puede listar cualquier [facade](/docs/{{version}}/facades) que desee registrar:

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
    

Una vez que su paquete ha sido configurado para ser descubierto, Laravel registrará automáticamente sus *service providers* y *facades* cuando se instale, creando una experiencia de instalación sencillísima para los usuarios de su paquete.

### Optar por no descubrir paquetes

Si usted es el consumidor de un paquete y desea deshabilitar esta característica, puede listar el nombre del paquete en la sección `extra` del archivo `composer.json` de su aplicación:

    "extra": {
        "laravel": {
            "dont-discover": [
                "barryvdh/laravel-debugbar"
            ]
        }
    },
    

Puede deshabilitarlo para todos los paquetes utilizando el caracter `*` dentro de la directiva `dont-discover` de su aplicación:

    "extra": {
        "laravel": {
            "dont-discover": [
                "*"
            ]
        }
    },
    

<a name="service-providers"></a>

## Service Providers

Los [Service Providers](/docs/{{version}}/providers) son el punto de conexión entre el paquete y Laravel. Un *service provider* es responsable de incluir cosas en el [service container](/docs/{{version}}/container) de Laravel e informar al framework de donde cargar recursos como vistas, configuración o archivos de idioma.

Un *service provider* hereda de la clase `Illuminate\Support\ServiceProvider` y contiene dos métodos: `register` y `boot`. La clase base `ServiceProvider` se encuentra en el paquete de Composer `illuminate/support`, el cual debería agregarse a sus dependencias del propio paquete. Para obtener más información sobre la estructura y el propósito de los *service providers*, consulte [su documentación](/docs/{{version}}/providers).

<a name="resources"></a>

## Recursos – *Resources*

<a name="configuration"></a>

### Configuración

Normalmente, necesitará publicar el archivo de configuración de su paquete en el propio directorio `config` de la aplicación. Esto permite a los usuarios modificar fácilmente las opciones por defecto de configuración del paquete. Para permitir que sus archivos de configuración se publiquen, llame al método `publishes` desde el método `boot` de su *service provider*:

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
    

Ahora, cuando los usuarios de su paquete ejecuten el comando `vendor:publish` de Laravel, su archivo se copiará en la ubicación de publicación especificada. Por supuesto, una vez que su configuración ha sido publicada, se puede acceder a sus valores como cualquier otro archivo de configuración:

    $value = config('courier.option');
    

> {note} No debe definir *Closures* en los archivos de configuración. No se pueden "serializar" correctamente cuando los usuarios ejecutan el comando Artisan `config:cache`.

#### Configuración de paquetes por defecto

También puede fusionar su propio archivo de configuración de paquetes con la copia publicada de la aplicación. Esto permitirá a sus usuarios definir sólo las opciones que realmente desean anular en la copia publicada de la configuración. Para combinar las configuraciones, se usa el método `mergeConfigFrom` dentro del método `register` del *service provider*:

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
    

> {note} Este método sólo fusiona el primer nivel del *array* de configuración. Si sus usuarios definen parcialmente un *array* de configuración multidimensional, las opciones que faltan no se fusionarán.

<a name="routes"></a>

### Rutas

Si su paquete contiene rutas, puede cargarlas usando el método `loadRoutesFrom`. Este método determinará automáticamente si las rutas de la aplicación están almacenadas en caché y no cargará el archivo de ser así:

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

### Migraciones

Si su paquete contiene [migraciones](/docs/{{version}}/migrations), puede utilizar el método `loadMigrationsFrom` para informar a Laravel cómo cargarlas. El método `loadMigrationsFrom` acepta la ruta a las migraciones de su paquete como su único argumento:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadMigrationsFrom(__DIR__.'/path/to/migrations');
    }
    

Una vez registradas las migraciones de su paquete, se ejecutarán automáticamente cuando se ejecute el comando `php artisan migrate`. No es necesario exportarlos al directorio principal de la aplicación `database/migrations`.

<a name="translations"></a>

### Traducciones

Si el paquete contiene [archivos de traducción](/docs/{{version}}/localization), tiene disponible el método `loadTranslationsFrom` para informar a Laravel como cargarlos. Por ejemplo, si su paquete se llama `courier`, debería añadir lo siguiente al método `boot` de su *service provider*:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
    }
    

Las traducciones de paquetes son referenciadas usando la convención de sintaxis `paquete::file.line`. Por lo tanto, se puede cargar del paquete `courier` la línea `welcome` del archivo `messages` de la siguiente forma:

    echo trans('courier::messages.welcome');
    

#### Publicar traducciones

Si se quiere publicar las traducciones de un paquete dentro del directorio `resources/lang/vendor` de la aplicación, se puede usar el método `publishes` del *service provider*. El método `publishes` acepta un *array* de rutas de paquetes y sus ubicaciones de publicación deseadas. Por ejemplo, para publicar los archivos de traducción del paquete `courier` puede hacer lo siguiente:

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
    

Ahora, cuando los usuarios de su paquete ejecuten el comando Artisan `vendor:publish` de Laravel, las traducciones de su paquete se publicarán en la ubicación de publicación especificada.

<a name="views"></a>

### Vistas

Para registrar las [vistas](/docs/{{version}}/views) del paquete con Laravel, es necesario informar al framework donde se encuentran. Esto se puede hacer utilizando el método `loadViewsFrom` del *service provider*. El método `loadViewsFrom` acepta dos argumentos: la ruta a las vistas y el nombre del paquete. Por ejemplo, si el nombre de su paquete es `courier`, puede añadir lo siguiente al método `boot` de su *service provider*:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
    }
    

Las vistas de paquetes se referencian usando la convención de sintaxis `package::view`. Por lo tanto, una vez que su ruta de vista está registrada en un *service provider*, puede cargar la vista `admin` desde el paquete `courier`:

    Route::get('admin', function () {
        return view('courier::admin');
    });
    

#### Sobrescribir las vistas de un paquete

Cuando utiliza el método `loadViewsFrom`, Laravel registra dos ubicaciones para sus vistas: el directorio `resources/views/vendor` de la aplicación y el directorio que especifique. Por lo tanto, usando el ejemplo `courier`, Laravel primero comprobará si el desarrollador ha proporcionado una versión personalizada de la vista en `resources/views/vendor/vendor/courier`. Si no se encuentra, Laravel buscará la vista en el directorio que se especificó en el método `loadViewsFrom`. Esto hace que sea fácil para los usuarios de paquetes personalizar / anular las vistas de su paquete.

#### Publicar las vistas

Si se quiere publicar las vistas de un paquete dentro del directorio `resources/views/vendor` de la aplicación, se puede usar el método `publishes` del *service provider*. El método `publishes` acepta un conjunto de rutas de vista de paquetes y sus ubicaciones de publicación deseadas:

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
    

Ahora, cuando los usuarios de su paquete ejecuten el comando Artisan `vendor:publish` de Laravel, las vistas de su paquete se copiarán en la ubicación de publicación especificada.

<a name="commands"></a>

## Commandos

Para registrar los comandos Artisan de su paquete con Laravel, puede usar el método `commands`. Este método espera un *array* de nombres de clases de comando. Una vez registrados los comandos, puede ejecutarlos utilizando [Artisan CLI](/docs/{{version}}/artisan):

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

## Publicar *assets*

Su paquete puede tener recursos como JavaScript, CSS e imágenes. Para publicar estos recursos dentro del directorio `public` de la aplicación, se utiliza el método `publishes` del *service provider*. En este ejemplo, añadiremos también una etiqueta `public`, la cual puede utilizarse para publicar grupos de recursos relacionados:

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
    

Ahora, cuando los usuarios del paquete ejecuten el comando `vendor:publish`, sus activos se copiarán en la ubicación de publicación especificada. Como normalmente necesitará sobrescribir los activos cada vez que actualice el paquete, puede utilizar el indicador `--force`:

    php artisan vendor:publish --tag=public --force
    

<a name="publishing-file-groups"></a>

## Publicar grupos de ficheros

Puedes querer publicar un grupo de recursos del paquete de forma separada. Por ejemplo, es posible que desee permitir a sus usuarios publicar los archivos de configuración de su paquete sin verse forzado a publicar los recursos. Puede hacerlo "etiquetándolos" cuando llame al método `publishes` del proveedor de servicios de un paquete. Por ejemplo, usemos etiquetas para definir dos grupos de publicación en el método `boot` de un *service provider* de un paquete:

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
    

Ahora los usuarios pueden publicar estos grupos por separado haciendo referencia a su etiqueta al ejecutar el comando `vendor:publish`:

    php artisan vendor:publish --tag=config