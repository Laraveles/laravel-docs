# Redis

- [Introducción](#introduction) 
    - [Configuración](#configuration)
    - [Predis](#predis)
    - [PhpRedis](#phpredis)
- [Interactuando con Redis](#interacting-with-redis) 
    - [Tuberías de comandos](#pipelining-commands)
- [Pub / Sub](#pubsub)

<a name="introduction"></a>

## Introducción

[Redis](https://redis.io) es un sistema de almacenamiento avanzado clave-valor de código abierto. Normalmente se refiere a el como un servidor de estructuras de datos puesto que las claves pueden contener [strings](https://redis.io/topics/data-types#strings), [hashes](https://redis.io/topics/data-types#hashes), [listas](https://redis.io/topics/data-types#lists), [sets](https://redis.io/topics/data-types#sets), y [sets ordenados](https://redis.io/topics/data-types#sorted-sets).

Antes de utilizar Redis con Laravel, es necesario instalar el paquete `predis/predis` a través de Composer:

    composer require predis/predis
    

Alternativamente, se puede instalar la extensión de PHP [PhpRedis](https://github.com/phpredis/phpredis) vía PECL. La extensión es más compleja de instalar pero puede obtener mejor rendimiento para aplicaciones que tengan un uso intensivo de Redis.

<a name="configuration"></a>

### Configuración

La configuración de Redis para una aplicación se encuentra en el archivo de configuración `config/database.php`. En este archivo, verá un *array* `redis` que contiene todos los servidores Redis utilizados por la aplicación:

    'redis' => [
    
        'client' => 'predis',
    
        'default' => [
            'host' => env('REDIS_HOST', 'localhost'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', 6379),
            'database' => 0,
        ],
    
    ],
    

La configuración del servidor por defecto debería ser suficiente para el desarrollo. Sin embargo, se puede modificar este *array* dependiendo de su entorno. Cada servidor Redis definido en la configuración precisa de un nombre, *host* y puerto.

#### Configurar Clusters

Si su aplicación utiliza un cluster de servidores Redis, se debe definir en la clave `clusters` de la configuración de Redis:

    'redis' => [
    
        'client' => 'predis',
    
        'clusters' => [
            'default' => [
                [
                    'host' => env('REDIS_HOST', 'localhost'),
                    'password' => env('REDIS_PASSWORD', null),
                    'port' => env('REDIS_PORT', 6379),
                    'database' => 0,
                ],
            ],
        ],
    
    ],
    

By default, clusters will perform client-side sharding across your nodes, allowing you to pool nodes and create a large amount of available RAM. However, note that client-side sharding does not handle failover; therefore, is primarily suited for cached data that is available from another primary data store. Si quiere utilizar los cluster de Redis nativos, debe especificarlo en la clave `options` del archivo de configuración de Redis:

    'redis' => [
    
        'client' => 'predis',
    
        'options' => [
            'cluster' => 'redis',
        ],
    
        'clusters' => [
            // ...
        ],
    
    ],
    

<a name="predis"></a>

### Predis

Además de las opciones de configuración del servidor por defecto `host`, `port`, `database`, y `password`, Predis soporta [parámetros de conexión](https://github.com/nrk/predis/wiki/Connection-Parameters) adicionales que pueden definirse para cada uno de los servidores Redis. Para utilizar estas opciones de configuración adicionales, simplemente añádalas a la configuración de servidor de Redis en el archivo de configuración `config/database.php`:

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_write_timeout' => 60,
    ],
    

<a name="phpredis"></a>

### PhpRedis

> {note} Si tiene la extensión de PhpRedis instalada via PECL, será necesario renombrar el alias `Redis` en el archivo de configuración `config/app.php`.

Para utilizar la extensión PhpRedis, debe cambiar la opción `client` de la configuración Redis a `phpredis`. Esta opción se encuentra en el archivo de configuración `config/database.php`:

    'redis' => [
    
        'client' => 'phpredis',
    
        // Rest of Redis configuration...
    ],
    

Además de las opciones de configuración del servidor por defecto `host`, `port`, `database`, y `password`, PhpRedis soporta los siguientes parámetros de conexión: `persistent`, `prefix`, `read_timeout` and `timeout`. Se puede añadir cualquiera de estas opciones a la configuración del servidor Redis en el archivo de configuración `config/database.php`:

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_timeout' => 60,
    ],
    

<a name="interacting-with-redis"></a>

## Interactuar con Redis

Se puede interactuar con Redis llamado varios métodos de la [facade](/docs/{{version}}/facades) `Redis`. La *facade* `Redis` soporta métodos dinámicos, por lo que se puede llamar a cualquier [comando Redis](https://redis.io/commands) en la *facade</1> y este se pasará directamente a Redis. En este ejemplo, se ejecutará el comando `GET` de Redis utilizando el método `get` de la *facade</code> `Redis</0>:</p>

<pre><code><?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Redis;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     *
     * @param  int  $id
     * @return Response
     */
    public function showProfile($id)
    {
        $user = Redis::get('user:profile:'.$id);

        return view('user.profile', ['user' => $user]);
    }
}
`</pre> 

Por supuesto, como se mencionó anteriormente, se puede llamar cualquier comando Redis dentro de la *facade* `Redis`. Laravel usa los métodos mágicos para pasar los comandos al servidor Redis, así que simplemente basta con pasar los argumentos que espera el comando Redis:

    Redis::set('name', 'Taylor');
    
    $values = Redis::lrange('names', 5, 10);
    

Como otra alternativa, se puede pasar el comando al servidor usando el método `command`, el mismo acepta el nombre del comando como primer argumento y un *array* de valores como segundo argumento:

    $values = Redis::command('lrange', ['name', 5, 10]);
    

#### Utilizar varias conexiones de Redis

Se puede obtener una instancia de Redis llamando el método `Redis::connection`:

    $redis = Redis::connection();
    

Esto retorna una instancia del servidor Redis predeterminado. También se puede pasar el nombre de una conexión o cluster al método `connection` para obtener un servidor o cluster específico tal y como se ha definido en su archivo de configuración Redis:

    $redis = Redis::connection('my-connection');
    

<a name="pipelining-commands"></a>

### Tuberías de comandos

Se deben utilizar tuberías cuando se desee enviar varios comandos al servidor en una misma operación. El método `pipeline` acepta un argumento: un `Closure` que recibe una instancia de Redis. Se pueden enviar todos sus comandos a esta instancia de Redis y se ejecutarán en una única operación:

    Redis::pipeline(function ($pipe) {
        for ($i = 0; $i < 1000; $i++) {
            $pipe->set("key:$i", $i);
        }
    });
    

<a name="pubsub"></a>

## Pub / Sub

Laravel provee una interfaz para los comandos de Redis `publish` y `subscribe`. Estos comandos de Reids permiten escuchar mensajes en un "canal" concreto. Se pueden publicar mensajes en el canal desde otra aplicación o incluso desde otro lenguaje de programación, permitiendo una comunicación sencilla entre aplicaciones y procesos.

Primero hay que configurar un *listener* de canal utilizando el método `subscribe`. Este método se ubicará en un [comando Artisan](/docs/{{version}}/artisan) puesto que ejecutar el método `subscribe` comienza un proceso largo:

    <?php
    
    namespace App\Console\Commands;
    
    use Illuminate\Console\Command;
    use Illuminate\Support\Facades\Redis;
    
    class RedisSubscribe extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'redis:subscribe';
    
        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Subscribe to a Redis channel';
    
        /**
         * Execute the console command.
         *
         * @return mixed
         */
        public function handle()
        {
            Redis::subscribe(['test-channel'], function ($message) {
                echo $message;
            });
        }
    }
    

Llegados a este punto, ya se pueden publicar mensajes en el canal utilizando el método `publish`:

    Route::get('publish', function () {
        // Route logic...
    
        Redis::publish('test-channel', json_encode(['foo' => 'bar']));
    });
    

#### Subscripciones comodín

Utilizando el método `psubscribe`, se puede subscribir a un canal comodín, el cual puede ser útil para capturar todos los mensajes de todos los canales. El nombre de `$channel` se pasará como segundo argument al `Closure` proporcionado como *callback*:

    Redis::psubscribe(['*'], function ($message, $channel) {
        echo $message;
    });
    
    Redis::psubscribe(['users.*'], function ($message, $channel) {
        echo $message;
    });