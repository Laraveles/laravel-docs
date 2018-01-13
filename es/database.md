# Bases de datos: Comienzo

- [Introducción](#introduction) 
    - [Configuración](#configuration)
    - [Conexiones lectura & escritura](#read-and-write-connections)
    - [Usando varias conexiones de base de datos](#using-multiple-database-connections)
- [Ejecutar consultas en SQL](#running-queries) 
    - [Capturar eventos de consultas](#listening-for-query-events)
- [Transacciones de la base de datos](#database-transactions)

<a name="introduction"></a>

## Introducción

Laravel realiza interacciones con bases de datos y ejecuta consultas de forma muy simple a través de varias configuraciones de base de datos utilizando SQL puro, el [generador de consultas Fluent](/docs/{{version}}/queries) y el [ORM Eloquent](/docs/{{version}}/eloquent). Actualmente, Laravel soporta cuatro sistemas de bases de datos:

<div class="content-list">
  <ul>
    <li>
      MySQL
    </li>
    <li>
      PostgreSQL
    </li>
    <li>
      SQLite
    </li>
    <li>
      SQL Server
    </li>
  </ul>
</div>

<a name="configuration"></a>

### Configuración

La configuración de la base de datos para la aplicación se encuentra en `config/database.php`. En este archivo se pueden definir todas las conexiones de base de datos, así como especificar qué conexión se debe utilizar por defecto. Este archivo proporciona ejemplos para la mayoría los sistemas de base de datos soportados.

Por defecto, la [configuración de entorno](/docs/{{version}}/configuration#environment-configuration) de ejemplo de Laravel está preparada para ser usada con [Laravel Homestead](/docs/{{version}}/homestead), que es una máquina virtual adecuada para desarrollar con Laravel en un equipo en local. Por supuesto, hay libertad para modificar esta configuración cuando sea necesario para una base de datos en local.

#### Configuración de SQLite

Después de crear una nueva base de datos SQLite usando un comando como `touch database/database.sqlite`, se puede configurar fácilmente las variables de entorno para que apunten a esta base de datos recién creada utilizando la ruta absoluta de la base de datos:

    DB_CONNECTION=sqlite
    DB_DATABASE=/absolute/path/to/database.sqlite
    

<a name="read-and-write-connections"></a>

### Conexiones lectura & escritura

A veces se puede usar una conexión a una base de datos para las sentencias SELECT y otra conexión para las sentencias INSERT, UPDATE y DELETE. Laravel hace las cosas sencillas, y se usarán las conexiones adecuadas tanto si se utilizan consultas en SQL puro, el generador de consultas o el ORM Eloquent.

En el siguiente ejemplo, se muestra como deberían estar configuradas las conexiones de lectura / escritura:

    'mysql' => [
        'read' => [
            'host' => '192.168.1.1',
        ],
        'write' => [
            'host' => '196.168.1.2'
        ],
        'sticky'    => true,
        'driver'    => 'mysql',
        'database'  => 'database',
        'username'  => 'root',
        'password'  => '',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_unicode_ci',
        'prefix'    => '',
    ],
    

Hay que observar que se han añadido tres claves a la configuración del *array*: `read`, `write` y `sticky`. Las claves `read` y `write` contienen un *array* con una sola clave: `host`. El resto de las opciones de base de datos para las conexiones de `read` y `write` se mezclarán desde el *array* principal `mysql`.

De esta forma, solo se tienen que colocar elementos en `read` y `write` si se desea sobrescribir los valores del *array* principal. Por lo tanto, en este caso, `192.168.1.1` será utilizado para la conexión "read" mientras que `192.168.1.2` será utilizado para la conexión "write". Las credenciales, prefijos, configuración de caracteres y el resto de las opciones del *array* principal `mysql` serán compartidas por ambas conexiones.

#### La opción `sticky`

La opción `sticky` es un valor *optional* que puede ser usado para permitir la lectura inmediata de grabaciones que han sido escritas en la base de datos durante el ciclo de solicitud actual. Si la opción `sticky`está habilitada y se ha realizado una operación de "escritura" contra la base de datos durante el ciclo de solicitud actual, cualquier operación adicional de "lectura" utilizará la conexión de "escritura". Esto garantiza que cualquier dato escrito durante el ciclo de solicitud pueda leerse inmediatamente desde la base de datos durante la misma solicitud. Depende de usted decidir si este es el comportamiento deseado para su aplicación.

<a name="using-multiple-database-connections"></a>

### Usando múltiples conexiones de base de datos

Cuando se usan múltiples conexiones, se puede acceder a cada conexión con el método `connection` de la fachada `DB`. El `nombre` pasado al método `connection` debería corresponderse con uno de las conexiones listadas en el archivo de configuración `config/database.php`:

    $users = DB::connection('foo')->select(...);
    

Se puede acceder a la instancia PDO subyacente utilizando el método `getPdo` en la instancia de una conexión:

    $pdo = DB::connection()->getPdo();
    

<a name="running-queries"></a>

## Ejecutar Consultas en SQL Puro

Una vez configurada la conexión a la base de datos, se pueden ejecutar consultas usando la *facade* `DB`. La *facade* `DB` proporciona métodos para cada tipo de consulta: `select`, `update`, `insert`, `delete`, y `statement`.

#### Ejecutando Una Consulta Select

Para ejecutar una consulta básica, se puede utilizar el método `select` de la *facade* `DB`:

    <?php
    
    namespace App\Http\Controllers;
    
    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;
    
    class UserController extends Controller
    {
        /**
         * Show a list of all of the application's users.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::select('select * from users where active = ?', [1]);
    
            return view('user.index', ['users' => $users]);
        }
    }
    

El primer argumento pasado al método `select` es la consulta en SQL puro, mientras que el segundo argumento son los enlaces a los parámetros considerados obligatorios en la consulta. En general, estos son los valores de las restricciones de la cláusula `where`. Los parámetros enlazados proporcionan protección contra la inyección SQL.

El método `select` siempre devolverá un `<em>array</em>` de resultados. Cada resultado dentro del *array* será un objeto de PHP `StdClass`, que permitirá acceder a los valores de los resultados:

    foreach ($users as $user) {
        echo $user->name;
    }
    

#### Uso de Enlaces con Nombre

En vez de usar `?` para representar enlaces a los parámetros, se puede utilizar una consulta utilizando enlaces con nombres:

    $results = DB::select('select * from users where id = :id', ['id' => 1]);
    

#### Ejecución de una Sentencia de Inserción

Para ejecutar una sentencia `insert`, se debe utilizar el método `insert` de la *facade* `DB`. Como en el `select`, este método tiene la consulta en SQL puro como primer parámetro y los enlaces como segundo argumento:

    DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);
    

#### Ejecución de una Sentencia Update

El método `update` se utiliza para actualizar los registros existentes en la base de datos. El método devolverá el número de filas afectadas por la sentencia:

    $affected = DB::update('update users set votes = 100 where name = ?', ['John']);
    

#### Ejecición de una Sentencia Delete

El método `delete` se debería usar para borrar registros de la base de datos. Al igual que `update`, devolverá el número de filas eliminadas:

    $deleted = DB::delete('delete from users');
    

#### Ejecución de una Sentencia General

Algunas sentencias de bases de datos no devuelven ningún valor. Para este tipo de operaciones, se debe utilizar el método `statement` de la *facade* `DB`:

    DB::statement('drop table users');
    

<a name="listening-for-query-events"></a>

### Capturar Eventos de Consultas

Si se desea recibir cada consulta SQL que es ejecutada en la aplicación, se debe usar el método `listen`. Este método es útil para realizar el registro o la depuración de consultas. Se puede registrar un capturador de consultas en un [service provider](/docs/{{version}}/providers):

    <?php
    
    namespace App\Providers;
    
    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\ServiceProvider;
    
    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            DB::listen(function ($query) {
                // $query->sql
                // $query->bindings
                // $query->time
            });
        }
    
        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }
    

<a name="database-transactions"></a>

## Transacciones de la base de datos

Se puede usar el método `transaction` en la facde `DB` para ejecutar un conjunto de operaciones dentro de una transacción de base de datos. Si se lanza una excepción dentro del `Closure` de la transacción, esta realizará automáticamente un rollback. Si el `Closure` se ejecuta de forma satisfactoria, la transacción realizará automáticamente un commit. No hay que preocuparse de hacer manualmente rollbacks o commits mientras se utilice el método `transaction`:

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);
    
        DB::table('posts')->delete();
    });
    

#### Manejo de interbloqueos

El método `transaction` acepta un segundo argumento opcional que define el número de veces que una transacción debe ser re-intentada cuando ocurre un interbloqueo. Una vez que estos intentos se hayan agotado, se lanzará una excepción:

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);
    
        DB::table('posts')->delete();
    }, 5);
    

#### Usar Transacciones Manualmente

Si se desea comenzar una transacción manual y tener el control sobre rollbacks y commits, se debe utilizar el método `beginTransaction` de la *facade* `DB`:

    DB::beginTransaction();
    

Se puede hacer rollback en la transacción utilizando el método `rollBack`:

    DB::rollBack();
    

Por último, se puede hace un commit de la transacción utilizando el método `commit`:

    DB::commit();
    

> {tip} Los métodos de transacción del *facade* `DB` controlan las transacciones para el [query builder](/docs/{{version}}/queries) y [Eloquent ORM](/docs/{{version}}/eloquent).