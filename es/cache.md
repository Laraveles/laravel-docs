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
    - [Almacenar elementos etiquetados en caché](#storing-tagged-cache-items)
    - [Acceder a elementos etiquetados en caché](#accessing-tagged-cache-items)
    - [Eliminar elementos etiquetados en caché](#removing-tagged-cache-items)
- [Añadir *drivers* personalizados](#adding-custom-cache-drivers) 
    - [Escribir el *driver*](#writing-the-driver)
    - [Registrar el *driver*](#registering-the-driver)
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
    

También se puede establecer la opción `host` a un *socket* UNIX. De ser así, la opción `port` debe establecerse en `0`:

    'memcached' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],
    

#### Redis

Antes de utilizar Reids con Laravel, es necesario instalar el paquete `predis/predis` (~1.0) a través de Composer o instalar la exensión de PHP PhpRedis vía PECL.

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
    

#### Acceder a varios sistemas de caché

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

### Almacenar elementos en caché

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

### Almacenar elementos etiquetados en caché

Las etiquetas de caché permiten etiquetar elementos relacionados en la caché y luego eliminar todos los valores almacenados que han sido asignados a una etiqueta determinada. Se puede acceder a una caché etiquetada pasando un *array* ordenado de nombres de etiquetas. Por ejemplo, para acceder a una caché etiquetada y hacer `put` (almacenar) de un valor en la caché:

    Cache::tags(['people', 'artists'])->put('John', $john, $minutes);
    
    Cache::tags(['people', 'authors'])->put('Anne', $anne, $minutes);
    

<a name="accessing-tagged-cache-items"></a>

### Acceder a elementos etiquetados en caché

Para recuperar un elemento de caché etiquetado, pase la misma lista ordenada de etiquetas al método `tags` y llame al método `get` con la clave que desee recuperar:

    $john = Cache::tags(['people', 'artists'])->get('John');
    
    $anne = Cache::tags(['people', 'authors'])->get('Anne');
    

<a name="removing-tagged-cache-items"></a>

### Eliminar elementos etiquetados en caché

Se pueden eliminar todos los elementos asignados a una etiqueta o lista de etiquetas. Por ejemplo, esta declaración eliminaría todas las cachés etiquetadas con `people`, `authors` o ambos. Por lo que tanto `Anne` como `John` se eliminarían de la caché:

    Cache::tags(['people', 'authors'])->flush();
    

En contraste, esta declaración eliminaría sólo las cachés marcadas con `authors`, por lo que `Anne` se eliminaría, pero no `John`:

    Cache::tags('authors')->flush();
    

<a name="adding-custom-cache-drivers"></a>

## Añadir *drivers* personalizados

<a name="writing-the-driver"></a>

### Escribir el *driver*

Para crear un controlador de caché personalizado, es necesario implementar el [contrato](/docs/{{version}}/contracts) `Illuminate\Contracts\Cache\Store`. Por lo que una una implementación de caché con MongoDB tendría este aspecto:

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
    

Únicamente es necesario implementar cada uno de estos métodos utilizando una conexión MongoDB. Para ver un ejemplo de cómo implementar cada uno de estos métodos, revise `Illuminate\Cache\MemcachedStore` en el código fuente del framework. Una vez que la implementación esté completa, se puede registrar el *driver*personalizado</em>.

    Cache::extend('mongo', function ($app) {
        return Cache::repository(new MongoStore);
    });
    

> {tip} Si se está preguntando dónde colocar el código de su *driver* personalizado, puede crear un *namespace* llamado `Extensions` dentro de su directorio `app`. Tenga en cuenta que Laravel no posee una estructura de aplicación rígida y que se es libre de organizar la aplicación como de acuerdo con sus preferencias.

<a name="registering-the-driver"></a>

### Registrar el *driver*

Para registrar el *driver* de caché personalizado en Laravel puede utilizar el método `extend` en la *facade* `Cache`. La llamada a `Caché::extend` podría hacerse en el método `boot` de `App\Providers\AppServiceProvider` que se incluye con las nuevas aplicaciones de Laravel, o puede crear su propio *service provider* para alojar la extensión - no olvide registrar el *provider* en el *array* de *providers* del archivo`config/app.php`:

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
    

El primer argumento del método `extend` será el nombre del *driver*. Este además debe corresponder con la opción `driver` del archivo de archivo de configuración `config/cache.php`. El segundo argumento será un *Closure* que debe devolver una instancia de `Illuminate\Cache\Repository`. El *Closure* recibirá una instancia `$app`, que es una instancia del [service container](/docs/{{version}}/container).

Una vez que su extensión esté registrada, simplemente actualice la opción `driver` en el archivo de configuración `config/cache.php` con el nombre de su extensión.

<a name="events"></a>

## Eventos

Para ejecutar código en cada operación de caché, se pueden capturar los [eventos](/docs/{{version}}/events) lanzados por la caché. Normalmente, estos *listeners* se suelen ubicar en el `EventServiceProvider`:

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