# Cache

- [Configuración](#configuration) 
    - [Prerrequisitos del *driver*](#driver-prerequisites)
- [Uso de Caché](#cache-usage) 
    - [Obtener una instancia de Caché](#obtaining-a-cache-instance)
    - [Recuperar elementos de la Caché](#retrieving-items-from-the-cache)
    - [Almacenar elementos en Caché](#storing-items-in-the-cache)
    - [Quitar elementos de Caché](#removing-items-from-the-cache)
    - [El helper Caché](#the-cache-helper)
- [Etiquetas de Caché](#cache-tags) 
    - [Storing Tagged Cache Items](#storing-tagged-cache-items)
    - [Accessing Tagged Cache Items](#accessing-tagged-cache-items)
    - [Removing Tagged Cache Items](#removing-tagged-cache-items)
- [Adding Custom Cache Drivers](#adding-custom-cache-drivers) 
    - [Writing The Driver](#writing-the-driver)
    - [Registering The Driver](#registering-the-driver)
- [Eventos](#events)

<a name="configuration"></a>

## Configuración

Laravel incorpora un API unificado para varios sistemas de caché. La configuración de caché se encuentra en `config/cache.php`. En este archivo se puede especificar qué *driver* utilizar por defecto para la aplicación. Laravel soporta de serie los sistemas de caché más populares como [Memcached](https://memcached.org) y [Redis](https://redis.io).

El archivo de configuración de caché incluye además otras opciones, las cuales se encuentran documentadas y es recomendable leer. Por defecto, Laravel viene configurado para utilizar el *driver* `file`, el cual almacena objetos serializados en el sistema de archivos. Para aplicaciones más grandes, se recomienda utilizar un controlador más robusto como Memcached o Redis. Se pueden establecer varias configuraciones para un mismo *driver*.

<a name="driver-prerequisites"></a>

### Prerrequisitos del *driver*

#### Base de datos

Cuando se utiliza el *driver* `database`, es necesario crear una tabla que contendrá los elementos cacheados. Este es un ejemplo del `Schema` de esta tabla:

    Schema::create('cache', function ($table) {
        $table->string('key')->unique();
        $table->text('value');
        $table->integer('expiration');
    });
    

> {tip} También se puede usar el comando de Artisan `php artisan cache:table` para generar una migración con el esquema apropiado.

#### Memcached

El uso del controlador Memcached requiere la instalación del paquete [Memcached PECL](https://pecl.php.net/package/memcached). Puede listar todos sus servidores Memcached en el archivo de configuración `config/cache.php`:

    'memcached' => [
        [
            'host' => '127.0.0.1',
            'port' => 11211,
            'weight' => 100
        ],
    ],
    

También se puede establecer la opción `host` a un *socket* UNIX. De ser así, la opción `port` debe establecerse en ``:

    'memcached' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],
    

#### Redis

Before using a Redis cache with Laravel, you will need to either install the `predis/predis` package (~1.0) via Composer or install the PhpRedis PHP extension via PECL.

Para más información sobre cómo configurar Redis, consultar su [documentación en Laravel](/docs/{{version}}/redis#configuration).

<a name="cache-usage"></a>

## Uso de Caché

<a name="obtaining-a-cache-instance"></a>

### Obtener una Instancia de Caché

Los [contracts](/docs/{{version}}/contracts) `Illuminate\Contracts\Cache\Factory` y `Illuminate\Contracts\Cache\Repository` proporcionan acceso a los servicios de caché de Laravel. El contrato `Factory` permite acceder a todos los *drivers* de caché definidos en la aplicación. El contrato `Repository` es normalmente una implementación del *driver* de caché por defecto de la aplicación especificado por el archivo de configuración `cache`.

Sin embargo, también se puede utilizar la *facade* `Cache`, la cual se utilizará a lo largo de esta documentación. La *facade* `Cache` proporciona un acceso cómodo y preciso a las implementaciones subyacentes de los contratos de caché de Laravel:

    <?php
    
    namespace App\Http\Controllers;
    
    use Illuminate\Support\Facades\Cache;
    
    class UserController extends Controller
    {
        /**
         * Show a list of all users of the application.
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');
    
            //
        }
    }
    

#### Accessing Multiple Cache Stores

Utilizando la *facade* `Cache`, se puede acceder a los diferentes sistemas de caché utilizando el método `store`. La clave pasada al método `store` corresponderá con una de las listadas en el *array* `stores` del archivo de configuración `cache`:

    $value = Cache::store('file')->get('foo');
    
    Cache::store('redis')->put('bar', 'baz', 10);
    

<a name="retrieving-items-from-the-cache"></a>

### Recuperar elementos de la Caché

El método `get` de la *facade* `Cache` se utiliza para obtener elementos desde la caché. Si el elemento no existe en la caché, se retornará `null`. Si lo desea, puede pasar un segundo argumento al método `get` especificando el valor predeterminado que desea que se devuelva si el elemento no existe:

    $value = Cache::get('key');
    
    $value = Cache::get('key', 'default');
    

Se puede incluso pasar un `Closure` como valor por defecto. Se retornará el resultado de este `Closure` si el elemento especificado no existe en la caché. Pasar un *Closure* permite recuperar valores desde una base de datos o incluso un servicio externo:

    $value = Cache::get('key', function () {
        return DB::table(...)->get();
    });
    

#### Comprobar la existencia de un elemento

Se puede utilizar el método `has` para determinar si existe un elemento en la caché. Este método devuelve `false` si el valor es `null` o `false`:

    if (Cache::has('key')) {
        //
    }
    

#### Incrementar/decrementar valores

Los métodos `increment` y `decrement` se utilizan para ajustar el valor de elementos enteros en la caché. Ambos métodos aceptan un segundo argumento opcional que indica la cantidad por la cual se debe incrementar o disminuir el valor del elemento:

    Cache::increment('key');
    Cache::increment('key', $amount);
    Cache::decrement('key');
    Cache::decrement('key', $amount);
    

#### Recuperar & almacenar

A veces se desea recuperar un elemento de la caché, pero a su vez almacenar un valor por defecto si ese elemento no existe. Por ejemplo, obtener todos los usuarios desde la caché o, si no existen, recuperarlos de la base de datos y añadirlos a la caché. Para ello se puede utilizar el método `Cache::remember`:

    $value = Cache::remember('users', $minutes, function () {
        return DB::table('users')->get();
    });
    

Si el elemento no existe en la caché, el `Closure` pasado al método `remember` se ejecutará y se almacenará su resultado.

Puede utilizar el método `rememberForever` para recuperar un elemento de la caché o guardarlo para siempre:

    $value = Cache::rememberForever('users', function() {
        return DB::table('users')->get();
    });
    

#### Recuperar & borrar

Si necesita recuperar un elemento de la caché y luego eliminarlo, puede utilizar el método `pull`. Como el método `get`, retornará `null` si el elemento no existe:

    $value = Cache::pull('key');
    

<a name="storing-items-in-the-cache"></a>

### Storing Items In The Cache

Para almacenar elementos en la caché se puede utilizar el método `put` de la *facade* `Cache`. Cuando se coloca un elemento en la caché, es necesario especificar el número de minutos que el valor debe estar almacenado:

    Cache::put('key', 'value', $minutes);
    

En lugar de pasar el número de minutos como un entero, también puede pasar una instancia `DateTime` representando la hora/fecha de caducidad del elemento almacenado en caché:

    $expiresAt = Carbon::now()->addMinutes(10);
    
    Cache::put('key', 'value', $expiresAt);
    

#### Almacenar si no está presente

El método `add` únicamente añadirá un elemento a la caché si no existe previamente. Este método retornará `true` si el elemento ya existe en la caché. De otro modo, retornará `false`:

    Cache::add('key', 'value', $minutes);
    

#### Almacenar elementos indefinidamente

El método `forever` puede utilizarse para almacenar un elemento en la caché permanentemente. Dado que estos elementos no caducarán, deben eliminarse manualmente de la caché utilizando el método `forget`:

    Cache::forever('key', 'value');
    

> {tip} Si está utilizando el driver Memcached, los elementos que se almacenan "para siempre" pueden eliminarse cuando la caché alcanza su límite de tamaño.

<a name="removing-items-from-the-cache"></a>

### Borrar elementos de la Caché

Puede eliminar elementos de la caché utilizando el método `forget`:

    Cache::forget('key');
    

Y además se puede limpiar por completo la caché utilizando el método `flush`:

    Cache::flush();
    

> {note} La purga de la caché no respeta el prefijo de caché y eliminará todas las entradas. Tenga mucha precaución al limpiar una caché que pueda estar siendo compartida por otras aplicaciones.

<a name="the-cache-helper"></a>

### El *helper* Caché

Además de utilizar la *facade* `Cache` o el [contrato caché](/docs/{{version}}/contracts), también puede utilizar la función global `cache` para recuperar y almacenar datos. Cuando se llama a la función `cache` con una cadena como único argumento, retornará el valor para esa clave:

    $value = cache('key');
    

Si se proporciona un *array* de pares clave/valor y un tiempo de caducidad a la función, almacenará los valores en la caché durante el tiempo especificado:

    cache(['key' => 'value'], $minutes);
    
    cache(['key' => 'value'], Carbon::now()->addSeconds(10));
    

> {tip} Al "testear" la llamada a la función global `cache`, puede utilizar el método `Cache::shouldReceive` como si estuviera [testeando un facade](/docs/{{version}}/mocking#mocking-facades).

<a name="cache-tags"></a>

## Etiquetas de Caché

> {note} Las etiquetas de caché no son compatibles cuando se utilizan los *drivers* de caché `file` o `database`. Además, cuando se utilizan varias etiquetas con cachés que se almacenan "forever", el rendimiento aumenta con *drivers* como `memcached`, que automáticamente purgan registros obsoletos.

<a name="storing-tagged-cache-items"></a>

### Storing Tagged Cache Items

Cache tags allow you to tag related items in the cache and then flush all cached values that have been assigned a given tag. You may access a tagged cache by passing in an ordered array of tag names. For example, let's access a tagged cache and `put` value in the cache:

    Cache::tags(['people', 'artists'])->put('John', $john, $minutes);
    
    Cache::tags(['people', 'authors'])->put('Anne', $anne, $minutes);
    

<a name="accessing-tagged-cache-items"></a>

### Accessing Tagged Cache Items

To retrieve a tagged cache item, pass the same ordered list of tags to the `tags` method and then call the `get` method with the key you wish to retrieve:

    $john = Cache::tags(['people', 'artists'])->get('John');
    
    $anne = Cache::tags(['people', 'authors'])->get('Anne');
    

<a name="removing-tagged-cache-items"></a>

### Removing Tagged Cache Items

You may flush all items that are assigned a tag or list of tags. For example, this statement would remove all caches tagged with either `people`, `authors`, or both. So, both `Anne` and `John` would be removed from the cache:

    Cache::tags(['people', 'authors'])->flush();
    

In contrast, this statement would remove only caches tagged with `authors`, so `Anne` would be removed, but not `John`:

    Cache::tags('authors')->flush();
    

<a name="adding-custom-cache-drivers"></a>

## Adding Custom Cache Drivers

<a name="writing-the-driver"></a>

### Writing The Driver

To create our custom cache driver, we first need to implement the `Illuminate\Contracts\Cache\Store` [contract](/docs/{{version}}/contracts). So, a MongoDB cache implementation would look something like this:

    <?php
    
    namespace App\Extensions;
    
    use Illuminate\Contracts\Cache\Store;
    
    class MongoStore implements Store
    {
        public function get($key) {}
        public function many(array $keys);
        public function put($key, $value, $minutes) {}
        public function putMany(array $values, $minutes);
        public function increment($key, $value = 1) {}
        public function decrement($key, $value = 1) {}
        public function forever($key, $value) {}
        public function forget($key) {}
        public function flush() {}
        public function getPrefix() {}
    }
    

We just need to implement each of these methods using a MongoDB connection. For an example of how to implement each of these methods, take a look at the `Illuminate\Cache\MemcachedStore` in the framework source code. Once our implementation is complete, we can finish our custom driver registration.

    Cache::extend('mongo', function ($app) {
        return Cache::repository(new MongoStore);
    });
    

> {tip} If you're wondering where to put your custom cache driver code, you could create an `Extensions` namespace within your `app` directory. However, keep in mind that Laravel does not have a rigid application structure and you are free to organize your application according to your preferences.

<a name="registering-the-driver"></a>

### Registering The Driver

To register the custom cache driver with Laravel, we will use the `extend` method on the `Cache` facade. The call to `Cache::extend` could be done in the `boot` method of the default `App\Providers\AppServiceProvider` that ships with fresh Laravel applications, or you may create your own service provider to house the extension - just don't forget to register the provider in the `config/app.php` provider array:

    <?php
    
    namespace App\Providers;
    
    use App\Extensions\MongoStore;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\Support\ServiceProvider;
    
    class CacheServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Cache::extend('mongo', function ($app) {
                return Cache::repository(new MongoStore);
            });
        }
    
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }
    

The first argument passed to the `extend` method is the name of the driver. This will correspond to your `driver` option in the `config/cache.php` configuration file. The second argument is a Closure that should return an `Illuminate\Cache\Repository` instance. The Closure will be passed an `$app` instance, which is an instance of the [service container](/docs/{{version}}/container).

Once your extension is registered, simply update your `config/cache.php` configuration file's `driver` option to the name of your extension.

<a name="events"></a>

## Events

To execute code on every cache operation, you may listen for the [events](/docs/{{version}}/events) fired by the cache. Typically, you should place these event listeners within your `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Cache\Events\CacheHit' => [
            'App\Listeners\LogCacheHit',
        ],
    
        'Illuminate\Cache\Events\CacheMissed' => [
            'App\Listeners\LogCacheMissed',
        ],
    
        'Illuminate\Cache\Events\KeyForgotten' => [
            'App\Listeners\LogKeyForgotten',
        ],
    
        'Illuminate\Cache\Events\KeyWritten' => [
            'App\Listeners\LogKeyWritten',
        ],
    ];