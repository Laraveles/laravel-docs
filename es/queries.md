# Base de datos: *Query Builder*

- [Introducción](#introduction)
- [Obtener resultados](#retrieving-results) 
    - [Fragmentar resultados](#chunking-results)
    - [Funciones agregadas](#aggregates)
- [Selecciones (Select)](#selects)
- [Raw Expressions](#raw-expressions)
- [Joins](#joins)
- [Unions](#unions)
- [Cláusulas *where*](#where-clauses) 
    - [Agrupar parámetros](#parameter-grouping)
    - [Sentencias *where exists*](#where-exists-clauses)
    - [Sentencias *JSON where*](#json-where-clauses)
- [Ordenar, agrupar, limitar & offset](#ordering-grouping-limit-and-offset)
- [Cláusulas condicionales](#conditional-clauses)
- [Inserts](#inserts)
- [Updates](#updates) 
    - [Actualizar columnas JSON](#updating-json-columns)
    - [Incrementar & decrementar](#increment-and-decrement)
- [Deletes](#deletes)
- [Bloqueos persistentes](#pessimistic-locking)

<a name="introduction"></a>

## Introducción

El constructor de consultas o *query builder* de Laravel es una interfaz conveniente y fluida para ejecutar consultas de base de datos. Este puede usarse para realizar la mayoría de las operaciones de bases de datos de la aplicación y funciona para todos los sistemas de bases de datos soportados.

El generador de consultas de Laravel utiliza el enlace de parámetros de PDO para proteger la aplicación contra los ataques de inyección de SQL. No es necesario limpiar las cadenas que se pasan como enlaces.

<a name="retrieving-results"></a>

## Obtener resultados

#### Obtener todas las filas de una tabla

Puede usar el método `table` de la facade `DB` para comenzar una consulta. El método `table` devuelve una instancia del generador de consultas fluida para la tabla dada, lo que le permite encadenar más restricciones a la consulta y finalmente obtener los resultados utilizando el método `get</ 0>:</p>

<pre><code><?php

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
        $users = DB::table('users')->get();

        return view('user.index', ['users' => $users]);
    }
}
`</pre> 

El método `get` devuelve una `Illuminate\Support\Collection` que contiene los resultados donde cada resultado es una instancia del objeto PHP ` StdClass`. Es posible acceder a cada valor de la columna mediante el acceso a la columna como una propiedad del objeto:

    foreach ($users as $user) {
        echo $user->name;
    }
    

#### Recuperando un Solo Registro / Columna desde una Tabla

Si solo necesita obtener una fila de una tabla de la base de datos, tu puedes utilizar el método `first`. este método devolverá un solo objeto de `StdClass`:

    $user = DB::table('users')->where('name', 'John')->first();
    
    echo $user->name;
    

Si usted no necesita el valor completo de una fila, puede obtener un solo valor del registro obtenido con el método `value`. Este método le permite obtener el valor directo de dicha columna:

    $email = DB::table('users')->where('name', 'John')->value('email');
    

#### Recuperando una lista de valores de una columna

Si deseas recuperar una Collection que contiene los valores de una sola columna, puedes usar el método `pluck`. En este ejemplo, recuperaremos una Collection con los títulos de la tabla roles:

    $titles = DB::table('roles')->pluck('title');
    
    foreach ($titles as $title) {
        echo $title;
    }
    

Puedes especificar una clave de columna personalizada para la Collection devuelta:

    $roles = DB::table('roles')->pluck('title', 'name');
    
    foreach ($roles as $name => $title) {
        echo $title;
    }
    

<a name="chunking-results"></a>

### Fragmentar Resultados

Si tu necesitas trabajar con miles de registros de base de datos, considera utilizar el metodo `chunk`. Este método recupera una pequeña porción o bloque de resultados cada vez en un `Closure` para su procesado. Este método es muy utilizado para crear [Artisan commands](/docs/{{version}}/artisan) que procesen miles de registros. Por ejemplo, vamos a trabajar con la tabla `users` en bloques de 100 registros a la vez:

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        foreach ($users as $user) {
            //
        }
    });
    

Puedes parar el proceso de bloques cuando están siendo procesados devolviendo `false` desde la `Closure`:

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        // Process the records...
    
        return false;
    });
    

<a name="aggregates"></a>

### Funciones agregadas

El constructor de consultas también provee una variedad de métodos agregados tales como `count`, `max`, `min`, `avg` y `sum`. Puedes llamar cualquiera de esos métodos después de construir tu consulta:

    $users = DB::table('users')->count();
    
    $price = DB::table('orders')->max('price');
    

Por supuesto, puedes combinar esos métodos con otras cláusulas para construir su consulta:

    $price = DB::table('orders')
                    ->where('finalized', 1)
                    ->avg('price');
    

<a name="selects"></a>

## Selecciones (Select)

#### Especificando una cláusula Select

Por supuesto, puede que no siempre quiera seleccionar todas las columnas de una tabla de la base de datos. Usando el método `select`, podrá especificar una cláusula a medida para la consulta:

    $users = DB::table('users')->select('name', 'email as user_email')->get();
    

El método `distinct` permite forzar la consulta para retornar resultados diferentes (elimina los que son iguales):

    $users = DB::table('users')->distinct()->get();
    

Si usted ya tiene una instancia del constructor de consultas, y desea agregar una columna a la cláusula Select existente, puede utilizar el método `addSelect`:

    $query = DB::table('users')->select('name');
    
    $users = $query->addSelect('age')->get();
    

<a name="raw-expressions"></a>

## Expresiones en crudo

Algunas veces necesitarás utilizar una expresión raw en una consulta. Para crear una expresión raw debes utilizar el método `DB::raw`:

    $users = DB::table('users')
                         ->select(DB::raw('count(*) as user_count, status'))
                         ->where('status', '<>', 1)
                         ->groupBy('status')
                         ->get();
    

> {note} Los estamentos raw se inyectarán en las consultas como cadenas, por lo que debes ser extremadamente cuidadoso para no crear vulnerabilidades de inyección SQL.

<a name="raw-methods"></a>

### Métodos raw

En lugar de utilizar `DB::raw`, puedes utilizar también los siguientes métodos para insertar expresiones raw en diversas partes de tus consultas:

#### `selectRaw`

El método `selectRaw` puede utilizarse en lugar de `select(DB::raw(...))`. Este método acepta un array opcional de enlaces como su segundo argumento:

    $orders = DB::table('orders')
                    ->selectRaw('price * ? as price_with_tax', [1.0825])
                    ->get();
    

#### `whereRaw / orWhereRaw`

Los métodos `whereRaw` y `orWhereRaw` pueden utilizarse para inyectar un cláusula raw `where` en tu consulta. Estos métodos aceptan un array de enlaces opcional como su segundo argumento:

    $orders = DB::table('orders')
                    ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
                    ->get();
    

#### `havingRaw / orHavingRaw`

Los métodos `havingRaw` y `orHavingRaw` pueden utilizarse para configurar una cadena raw como el valor de la clausula `having`:

    $orders = DB::table('orders')
                    ->select('department', DB::raw('SUM(price) as total_sales'))
                    ->groupBy('department')
                    ->havingRaw('SUM(price) > 2500')
                    ->get();
    

#### `orderByRaw`

El método `orderByRaw` puede usarse para configurar una cadena raw como el valor de la cláusula `order by`:

    $orders = DB::table('orders')
                    ->orderByRaw('updated_at - created_at DESC')
                    ->get();
    

<a name="joins"></a>

## Uniones

#### Inner Join Clause

El constructor de consultas también puede ser utilizado para escribir declaraciones de union (joins). Para realizar una consulta básica "inner join", puedes usar el método `join` en una instancia del constructor de consultas. El primer argumento pasado al método `join` es el nombre de la tabla que necesita unir, mientras que los argumentos restantes especifican las restricciones de columnas para la unión. Por supuest, como puede ver, puede unir múltiples tablas en una sola consulta:

    $users = DB::table('users')
                ->join('contacts', 'users.id', '=', 'contacts.user_id')
                ->join('orders', 'users.id', '=', 'orders.user_id')
                ->select('users.*', 'contacts.phone', 'orders.price')
                ->get();
    

#### Left Join Clause

Si quisiera realizar un "left join" en vez de un "inner join", utilice el método `leftjoin`. El método `leftjoin` tiene la misma firma que el método `join`:

    $users = DB::table('users')
                ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();
    

#### Cross Join Clause

Para realizar un "cross join" use el método `crossJoin` con el nombre de la tabla con la que quiere realizar una combinación cruzada. Las combinaciones cruzadas generan un producto cartesiano entre la primera tabla y la tabla unida:

    $users = DB::table('sizes')
                ->crossJoin('colours')
                ->get();
    

#### Advanced Join Clauses

Puede también especificar cláusulas de unión más avanzadas. Para empezar pasamos una `Closure` como segundo argumento en el método `join`. La `Closure` recibirá un objeto `JoinClause` el cual le permite especificar restricciones en la clausa `join`:

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
            })
            ->get();
    

Si quisiera usar una clausula estilo "where" en sus uniones, puede usar los métodos `where` y `orWhere` en una unión. En vez de comparar dos columnas, estos métodos compararán la columna contra un valor:

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')
                     ->where('contacts.user_id', '>', 5);
            })
            ->get();
    

<a name="unions"></a>

## Fusiones (Unions)

El constructor de consultas también provee una manera rápida de "unir" dos consultas. Por ejemplo, puedes crear una consulta inicial, y entonces utilizar el método `union` para fusionarla con la segunda consulta:

    $first = DB::table('users')
                ->whereNull('first_name');
    
    $users = DB::table('users')
                ->whereNull('last_name')
                ->union($first)
                ->get();
    

> {tip} El método `unionAll` también está disponible y tiene la misma forma que `union`.

<a name="where-clauses"></a>

## Cláusulas Where

#### Cláusulas Where Simples

Se puede usar el método `where` en una instancia del *query builder* para agregar clausulas de `where` a la consulta. La llamada más básica a `where` requiere de tres argumentos. El primer argumento es el nombre de la columna. El segundo argumento es un operador, el cuál puede ser cualquiera de los operadores soportados por la base de datos. Finalmente, el tercer argumento es el valor a evaluar contra la columna.

Por ejemplo, esta es una consulta que verifica el valor que la columna "votes" es igual a 100:

    $users = DB::table('users')->where('votes', '=', 100)->get();
    

Por conveniencia, si simplemente quiere verificar que la columna es igual a un valor dado, puede pasar el valor directamente como segundo parámetro al método `where`:

    $users = DB::table('users')->where('votes', 100)->get();
    

Por supuesto, puede usar una variedad de operadores cuando escribe cláusulas `where`:

    $users = DB::table('users')
                    ->where('votes', '>=', 100)
                    ->get();
    
    $users = DB::table('users')
                    ->where('votes', '<>', 100)
                    ->get();
    
    $users = DB::table('users')
                    ->where('name', 'like', 'T%')
                    ->get();
    

También puede pasar un array de condiciones a la función `where`:

    $users = DB::table('users')->where([
        ['status', '=', '1'],
        ['subscribed', '<>', '1'],
    ])->get();
    

#### Declaraciones O (Or)

Puede encadenar restricciones where, así como agregar cláusulas `or` a la consulta. El método `orWhere` acepta los mismos argumentos que el método `where`:

    $users = DB::table('users')
                        ->where('votes', '>', 100)
                        ->orWhere('name', 'John')
                        ->get();
    

#### Agregar cláusulas *Where* adicionales

**whereBetween**

El método `whereBetween` verifica que el valor de la columna se encuentra entre dos valores:

    $users = DB::table('users')
                        ->whereBetween('votes', [1, 100])->get();
    

**whereNotBetween**

El método `whereNotBetween` verifica que el valor de la columna se encuentre fuerade los dos valores:

    $users = DB::table('users')
                        ->whereNotBetween('votes', [1, 100])
                        ->get();
    

**whereIn / whereNotIn**

El método `whereIn` verifica que el valor de una columna dada se encuentre dentro de los valores del array:

    $users = DB::table('users')
                        ->whereIn('id', [1, 2, 3])
                        ->get();
    

El método `whereNotIn` verifica que el valor de una columna dada **no** se encuentre dentro de los valores del array:

    $users = DB::table('users')
                        ->whereNotIn('id', [1, 2, 3])
                        ->get();
    

**whereNull / whereNotNull**

El método `whereNull` verifica que el valor de la columna dada sea `NULL `:

    $users = DB::table('users')
                        ->whereNull('updated_at')
                        ->get();
    

El método `whereNotNull` verifica que el valor de la columna no sea `NULL `:

    $users = DB::table('users')
                        ->whereNotNull('updated_at')
                        ->get();
    

**whereDate / whereMonth / whereDay / whereYear**

El método `whereDate` se puede usar para comparar el valor de una columna con una fecha:

    $users = DB::table('users')
                    ->whereDate('created_at', '2016-12-31')
                    ->get();
    

El método `whereMonth` se puede usar para comparar el valor de una columna con un mes específico del año:

    $users = DB::table('users')
                    ->whereMonth('created_at', '12')
                    ->get();
    

El método `whereDay` se puede usar para comparar el valor de una columna con un día específico del mes:

    $users = DB::table('users')
                    ->whereDay('created_at', '31')
                    ->get();
    

El método `whereYear` se puede usar para comparar el valor de una columna con un año específico:

    $users = DB::table('users')
                    ->whereYear('created_at', '2016')
                    ->get();
    

**whereColumn**

El método `whereColumn` se puede usar para verificar que dos columnas son iguales:

    $users = DB::table('users')
                    ->whereColumn('first_name', 'last_name')
                    ->get();
    

También se puede pasar un operador de comparación al método:

    $users = DB::table('users')
                    ->whereColumn('updated_at', '>', 'created_at')
                    ->get();
    

Al método `whereColumn` también se le puede pasar un array con múltiples condiciones. Estas condiciones se unirán utilizando el operador `and`:

    $users = DB::table('users')
                    ->whereColumn([
                        ['first_name', '=', 'last_name'],
                        ['updated_at', '>', 'created_at']
                    ])->get();
    

<a name="parameter-grouping"></a>

### Agrupar parámetros

A veces se puede necesitar crear cláusulas where más avanzadas como cláusulas "where exists" o agrupaciones de parámetros anidados. El query builder de Laravel también puede manejar estos. To get started, let's look at an example of grouping constraints within parenthesis:

    DB::table('users')
                ->where('name', '=', 'John')
                ->orWhere(function ($query) {
                    $query->where('votes', '>', 100)
                          ->where('title', '<>', 'Admin');
                })
                ->get();
    

As you can see, passing a `Closure` into the `orWhere` method instructs the query builder to begin a constraint group. The `Closure` will receive a query builder instance which you can use to set the constraints that should be contained within the parenthesis group. The example above will produce the following SQL:

    select * from users where name = 'John' or (votes > 100 and title <> 'Admin')
    

<a name="where-exists-clauses"></a>

### Where Exists Clauses

The `whereExists` method allows you to write `where exists` SQL clauses. The `whereExists` method accepts a `Closure` argument, which will receive a query builder instance allowing you to define the query that should be placed inside of the "exists" clause:

    DB::table('users')
                ->whereExists(function ($query) {
                    $query->select(DB::raw(1))
                          ->from('orders')
                          ->whereRaw('orders.user_id = users.id');
                })
                ->get();
    

The query above will produce the following SQL:

    select * from users
    where exists (
        select 1 from orders where orders.user_id = users.id
    )
    

<a name="json-where-clauses"></a>

### JSON Where Clauses

Laravel also supports querying JSON column types on databases that provide support for JSON column types. Currently, this includes MySQL 5.7 and PostgreSQL. To query a JSON column, use the `->` operator:

    $users = DB::table('users')
                    ->where('options->language', 'en')
                    ->get();
    
    $users = DB::table('users')
                    ->where('preferences->dining->meal', 'salad')
                    ->get();
    

<a name="ordering-grouping-limit-and-offset"></a>

## Ordenación, Agrupamientos, Límites y Desplazamiento

#### orderBy

The `orderBy` method allows you to sort the result of the query by a given column. The first argument to the `orderBy` method should be the column you wish to sort by, while the second argument controls the direction of the sort and may be either `asc` or `desc`:

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->get();
    

#### latest / oldest

The `latest` and `oldest` methods allow you to easily order results by date. By default, result will be ordered by the `created_at` column. Or, you may pass the column name that you wish to sort by:

    $user = DB::table('users')
                    ->latest()
                    ->first();
    

#### inRandomOrder

The `inRandomOrder` method may be used to sort the query results randomly. For example, you may use this method to fetch a random user:

    $randomUser = DB::table('users')
                    ->inRandomOrder()
                    ->first();
    

#### groupBy / having

The `groupBy` and `having` methods may be used to group the query results. The `having` method's signature is similar to that of the `where` method:

    $users = DB::table('users')
                    ->groupBy('account_id')
                    ->having('account_id', '>', 100)
                    ->get();
    

For more advanced `having` statements, see the [`havingRaw`](#raw-methods) method.

#### skip / take

Para limitar el número de resultados devueltos por una consulta, o saltar un número dado de resultados en la misma, puedes utilizar los métodos `skip` y `take`:

    $users = DB::table('users')->skip(10)->take(5)->get();
    

Altenativamente, puedes usar los métodos `limit` y `offset`:

    $users = DB::table('users')
                    ->offset(10)
                    ->limit(5)
                    ->get();
    

<a name="conditional-clauses"></a>

## Conditional Clauses

Sometimes you may want clauses to apply to a query only when something else is true. For instance you may only want to apply a `where` statement if a given input value is present on the incoming request. You may accomplish this using the `when` method:

    $role = $request->input('role');
    
    $users = DB::table('users')
                    ->when($role, function ($query) use ($role) {
                        return $query->where('role_id', $role);
                    })
                    ->get();
    

The `when` method only executes the given Closure when the first parameter is `true`. If the first parameter is `false`, the Closure will not be executed.

You may pass another Closure as the third parameter to the `when` method. This Closure will execute if the first parameter evaluates as `false`. To illustrate how this feature may be used, we will use it to configure the default sorting of a query:

    $sortBy = null;
    
    $users = DB::table('users')
                    ->when($sortBy, function ($query) use ($sortBy) {
                        return $query->orderBy($sortBy);
                    }, function ($query) {
                        return $query->orderBy('name');
                    })
                    ->get();
    

<a name="inserts"></a>

## Inserciones

The query builder also provides an `insert` method for inserting records into the database table. The `insert` method accepts an array of column names and values:

    DB::table('users')->insert(
        ['email' => 'john@example.com', 'votes' => 0]
    );
    

You may even insert several records into the table with a single call to `insert` by passing an array of arrays. Each array represents a row to be inserted into the table:

    DB::table('users')->insert([
        ['email' => 'taylor@example.com', 'votes' => 0],
        ['email' => 'dayle@example.com', 'votes' => 0]
    ]);
    

#### Auto-Incrementing IDs

If the table has an auto-incrementing id, use the `insertGetId` method to insert a record and then retrieve the ID:

    $id = DB::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );
    

> {note} When using PostgreSQL the `insertGetId` method expects the auto-incrementing column to be named `id`. If you would like to retrieve the ID from a different "sequence", you may pass the column name as the second parameter to the `insertGetId` method.

<a name="updates"></a>

## Actualizaciones

Por supuesto, además de insertar registros en la base de datos, el *query builder* también puede actualizar los registros existentes usando el método `update`. El método `update`, como el método `insert`, acepta un array de columnas y pares de valores que contienen las columnas que se actualizarán. Puede restringir la consulta de `update` usando `where` en las cláusulas:

    DB::table('users')
                ->where('id', 1)
                ->update(['votes' => 1]);
    

<a name="updating-json-columns"></a>

### Actualizar columnas JSON

Al actualizar una columna JSON, se debe usar la sintaxis `->` para acceder a la clave adecuada en el objeto JSON. Esta operación solo se admite en bases de datos que admiten columnas JSON:

    DB::table('users')
                ->where('id', 1)
                ->update(['options->enabled' => true]);
    

<a name="increment-and-decrement"></a>

### Incrementar & decrementar

El generador de consultas también proporciona métodos convenientes para incrementar o disminuir el valor de una columna determinada. Esto es simplemente un acceso directo, que proporciona una interfaz más expresiva y concisa en comparación con la escritura manual de la declaración `update`.

Ambos métodos aceptan al menos un argumento: la columna para modificar. Y opcionalmente, se puede pasar un segundo argumento la cantidad por la cual la columna debe incrementarse o decrementarse:

    DB::table('users')->increment('votes');
    
    DB::table('users')->increment('votes', 5);
    
    DB::table('users')->decrement('votes');
    
    DB::table('users')->decrement('votes', 5);
    

También puede especificar columnas adicionales para actualizar durante la operación:

    DB::table('users')->increment('votes', 1, ['name' => 'John']);
    

<a name="deletes"></a>

## Borrados

El *query builder* también se puede usar para eliminar registros de la tabla a través del método `delete`. Se puede restringir las sentencias `delete` añadiendo `where` a las cláusulas antes de llamar al método `delete`:

    DB::table('users')->delete();
    
    DB::table('users')->where('votes', '>', 100)->delete();
    

Si desea truncar toda la tabla, lo que eliminará todas las filas y restablecerá el ID incremental a cero, se puede usar el método ` truncate `:

    DB::table('users')->truncate();
    

<a name="pessimistic-locking"></a>

## Bloqueos persistentes

The query builder also includes a few functions to help you do "pessimistic locking" on your `select` statements. To run the statement with a "shared lock", you may use the `sharedLock` method on a query. A shared lock prevents the selected rows from being modified until your transaction commits:

    DB::table('users')->where('votes', '>', 100)->sharedLock()->get();
    

Alternatively, you may use the `lockForUpdate` method. A "for update" lock prevents the rows from being modified or from being selected with another shared lock:

    DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();