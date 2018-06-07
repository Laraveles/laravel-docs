# Eloquent: Introducción

- [Introduccion](#introduction)
- [Definir Modelos](#defining-models) 
    - [Convenciones de Modelos Eloquent](#eloquent-model-conventions)
- [Retrieving Models](#retrieving-models) 
    - [Colecciones](#collections)
    - [Fragmentar Resultados](#chunking-results)
- [Obtener Modelos / Aggregates](#retrieving-single-models) 
    - [Obtener Aggregates](#retrieving-aggregates)
- [Insertar & Actualizar Modelos](#inserting-and-updating-models) 
    - [Inserciones](#inserts)
    - [Actualizaciones](#updates)
    - [Asignación Masiva (Mass Assignment)](#mass-assignment)
    - [Otros Metodos de Creación](#other-creation-methods)
- [Eliminar Modelos](#deleting-models) 
    - [Soft Deleting (Borrado Lógico)](#soft-deleting)
    - [Consultar Modelos Soft Deleted](#querying-soft-deleted-models)
- [Query Scopes](#query-scopes) 
    - [Global Scopes](#global-scopes)
    - [Local Scopes](#local-scopes)
- [Eventos](#events) 
    - [Observers](#observers)

<a name="introduction"></a>

## Introduccion

El ORM Eloquent que incluye Laravel provee una bonita y sencilla implementación de Active Record para trabajar con la base de datos. Cada tabla de la base de datos tiene un "Modelo" que se utiliza para interactuar con dicha tabla. Los modelos permiten consultar datos de las tablas, así como insertar nuevos elementos en ellas.

Antes de comenzar, hay que estar seguro de haber configurado la conexión a la base de datos en `config/database.php`. Para más información sobre como configurar la base de datos, consultar [la documentación](/docs/{{version}}/database#configuration).

<a name="defining-models"></a>

## Definir Modelos

Para comenzar, crearemos un modelo Eloquent. Los modelos se colocan normalmente en la raíz del directorio `app`, pero se pueden colocar en cualquier lugar que sea auto-cargado de acuerdo con el archivo `composer.json`. Todos los modelos Eloquent heredan de la clase `Illuminate\Database\Eloquent\Model`.

La forma más sencilla de crear un modelo es utilizando el [comando Artisan](/docs/{{version}}/artisan) `make:model`:

    php artisan make:model User
    

Para generar una [migración](/docs/{{version}}/migrations) cuando se genera el modelo, se puede utilizar la opción `--migration` o `-m`:

    php artisan make:model User --migration
    
    php artisan make:model User -m
    

<a name="eloquent-model-conventions"></a>

### Convenciones de Modelos Eloquent

Ahora, veamos un ejemplo de una clase modelo `Flight` (vuelo), el cual utilizaremos para obtener y almacenar información de la tabla `flights`:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Flight extends Model
    {
        //
    }
    

#### Nombres de Tablas

Tener en cuenta que no se le ha indicado a Eloquent qué tabla utilizar para el modelo `Flight`. Se utilizará el "snake case" del plural del nombre de la clase como nombre de la tabla de no especificarse otro explícitamente. Por lo tanto, en este caso, Eloquent asumirá que le modelo `Flight` almacena registros en la tabla `flights`. Se puede especificar otra tabla definiendo la propiedad `table` en el modelo:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Flight extends Model
    {
        /**
         * The table associated with the model.
         *
         * @var string
         */
        protected $table = 'my_flights';
    }
    

#### Claves Primarias

Eloquent también asumirá que cada tabla tiene una columna como clave primaria llamada `id`. Se puede definir la propiedad protegida `$primaryKey` para sobrescribir esta convención.

Además, Eloquent supone que la clave primaria es un valor entero creciente, lo que significa que, de manera predeterminada, la clave primaria se convertirá en `int` automáticamente. Si se desea utilizar una clave primaria no incremental o no numérica, se debe establecer la propiedad publica `$incrementing` en su modelo en `false`. Si la clave primaria no es un número entero, se debe establecer la propiedad protegida `$keyType` del modelo en `string`.

#### Timestamps

Por defecto, Eloquent espera que existan las columnas `created_at` y `updated_at` en las tablas. Para que Eloquent no gestione estas columnas automáticamente, establecer la propiedad `timestamps` del modelo a `false`:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Flight extends Model
    {
        /**
         * Indicates if the model should be timestamped.
         *
         * @var bool
         */
        public $timestamps = false;
    }
    

Para personalizar el formato de los timestamps, establecer la propiedad `dateFormat` del modelo. Esta propiedad determina como los atributos de fechas son almacenados en la base de datos, así como su formato cuando el modelo es serializado a un array o JSON:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Flight extends Model
    {
        /**
         * The storage format of the model's date columns.
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }
    

Si se necesita personalizar los nombres de las columnas utilizadas para almacenar los *timestamps*, se puede establecer las constantes `CREATED_AT` y `UPDATED_AT` en el modelo:

    <?php
    
    class Flight extends Model
    {
        const CREATED_AT = 'creation_date';
        const UPDATED_AT = 'last_update';
    }
    

#### Conexión de Base de Datos

Por defecto, todos los modelos Eloquent utilizarán la conexión por defecto configurada en la aplicación. Para especificar otra conexión para un modelo, utilizar la propiedad `$connection`:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Flight extends Model
    {
        /**
         * The connection name for the model.
         *
         * @var string
         */
        protected $connection = 'connection-name';
    }
    

<a name="retrieving-models"></a>

## Retrieving Models

Una vez que se ha creado un modelo y [su tabla asociada en la base de datos](/docs/{{version}}/migrations#writing-migrations), se puede comenzar a obtener datos de la base de datos. Cada modelo Eloquent es un potente [query builder](/docs/{{version}}/queries) que permite consultar de forma fluida la tabla de la base de datos asociada al modelo. Por ejemplo:

    <?php
    
    use App\Flight;
    
    $flights = App\Flight::all();
    
    foreach ($flights as $flight) {
        echo $flight->name;
    }
    

#### Añadir Constraints Adicionales

El método `all` de Eloquent retornará todos los resultados del la tabla del modelo. Puesto que cada modelo actúa como un [query builder](/docs/{{version}}/queries), se pueden añadir constraints a las consultas, y por tanto utilizar el método `get` para obtener los resultados:

    $flights = App\Flight::where('active', 1)
                   ->orderBy('name', 'desc')
                   ->take(10)
                   ->get();
    

> {tip} Nota: Puesto que los modelos Eloquent son query builders, se deben revisar todos los métodos disponibles en el [query builder](/docs/{{version}}/queries). Se puede utilizar cualquiera de estos métodos en Eloquent.

<a name="collections"></a>

### Colecciones

Para métodos de Eloquent como `all` y `get` que obtienen múltiples resultados, se retornará una instancia de `Illuminate\Database\Eloquent\Collection`. La clase `Collection` provee [una variedad de métodos útiles](/docs/{{version}}/eloquent-collections#available-methods) para trabajar los resultados de Eloquent:

    $flights = $flights->reject(function ($flight) {
        return $flight->cancelled;
    });
    

Por supuesto, se puede recorrer esta colección como un array:

    foreach ($flights as $flight) {
        echo $flight->name;
    }
    

<a name="chunking-results"></a>

### Fragmentar Resultados

Para procesar miles de registros Eloquent, utilizar el método `chunk`. El método `chunk` obtendrá "conjuntos" de modelos Eloquent alimentando con ellos un `Closure` para procesarlos. Utilizar el método `chunk` conservará memoria cuando se trabaja con grandes conjuntos de resultados:

    Flight::chunk(200, function ($flights) {
        foreach ($flights as $flight) {
            //
        }
    });
    

El primer argumento pasado el método es el número de registros que se desea recibir por "chunk". El Closure proporcionado como segundo parámetro se ejecutará por cada conjunto obtenido de la base de datos. Se ejecutará una consulta de base de datos para recuperar cada fragmento de registros pasados al *Clousure*.

#### Usando Cursores

El método `cursor` permite iterar a través de los registros de la base de datos usando un cursor, que solo ejecutará una sola consulta. Al procesar grandes cantidades de datos, el método `cursor` se puede usar para reducir en gran medida el uso de la memoria:

    foreach (Flight::where('foo', 'bar')->cursor() as $flight) {
        //
    }
    

<a name="retrieving-single-models"></a>

## Obtener Modelos / Aggregates

Por supuesto, además de obtener todos los registros de una tabla dada, se pueden obtener registros únicos utilizando `find` ó `first`. En lugar de devolver una colección de modelos, estos métodos retornan una instancia de un modelo único:

    // Retrieve a model by its primary key...
    $flight = App\Flight::find(1);
    
    // Retrieve the first model matching the query constraints...
    $flight = App\Flight::where('active', 1)->first();
    

También se puede llamar al método `find` con un array de claves primarias, que devolverá una colección de registros coincidentes:

    $flights = App\Flight::find([1, 2, 3]);
    

#### Excepciones Not Found

A veces se puede necesitar lanzar excepciones si un modelo no se encuentra. Esto resulta particularmente útil en rutas y controladores. Los métodos `findOrFail` y `firstOrFail` recuperarán el primer resultado de la consulta; sin embargo, si no se encuentra ningún resultado, se arrojará una `Illuminate\Database\Eloquent\ModelNotFoundException`:

    $model = App\Flight::findOrFail(1);
    
    $model = App\Flight::where('legs', '>', 100)->firstOrFail();
    

Si no se detecta la excepción, se envía automáticamente al usuario una respuesta HTTP `404`. No es necesario escribir comprobaciones explícitas para devolver respuestas `404 ` cuando se utilizan estos métodos:

    Route::get('/api/flights/{id}', function ($id) {
        return App\Flight::findOrFail($id);
    });
    

<a name="retrieving-aggregates"></a>

### Obtener Aggregates

Por supuesto, también se pueden utilizar los [métodos aggregate](/docs/{{version}}/queries#aggregates) `count`, `sum`, `max` y otras que provee el [query builder](/docs/{{version}}/queries). Estos métodos retornan el valor scalar apropiado en lugar de un modelo completo:

    $count = App\Flight::where('active', 1)->count();
    
    $max = App\Flight::where('active', 1)->max('price');
    

<a name="inserting-and-updating-models"></a>

## Insertar & Actualizar Modelos

<a name="inserts"></a>

### Inserciones

Para crear un nuevo registro en la base de datos, simplemente hay que crear una nueva instancia del modelo, establecer los atributos y llamar al método `call`:

    <?php
    
    namespace App\Http\Controllers;
    
    use App\Flight;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;
    
    class FlightController extends Controller
    {
        /**
         * Create a new flight instance.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Validate the request...
    
            $flight = new Flight;
    
            $flight->name = $request->name;
    
            $flight->save();
        }
    }
    

En este ejemplo, simplemente se asigna el parámetro `name` de la solicitud HTTP entrante al atributo `name` de la instancia del modelo `App\Flight`. Cuando se ejecuta el método `save` se insertará un registro en la base de datos. Los timestamps `created_at` y `updated_at` se establecen automáticamente cuando se llama al método `save`, por lo que no es necesario establecerlos manualmente.

<a name="updates"></a>

### Actualizaciones

El método `save` puede también utilizarse para actualizar modelos que ya existen en la base de datos. Para actualizar un modelo, se debe recuperar, establecer los atributos a actualizar, y llamar al método `save`. De nuevo, el timestamp `updated_at` se actualizará automáticamente, por lo que no es necesario establecer su valor manualmente:

    $flight = App\Flight::find(1);
    
    $flight->name = 'New Flight Name';
    
    $flight->save();
    

#### Mass Updates

Los updates se pueden ejecutar contra cualquier número de modelos que coincidan con un una consulta concreta. En este ejemplo, todos los vuelos que están `active` y tienen una `destination` de `San Diego` se marcarán como delayed (retrasado):

    App\Flight::where('active', 1)
              ->where('destination', 'San Diego')
              ->update(['delayed' => 1]);
    

El método `update` espera un array de pares de columnas y valores representando las columnas que se deben actualizar.

> {note} Cuando se ejecuta una declaración de actualización masiva a través de Eloquent, los eventos del modelo `saved` y `updated` no se dispararán. This is because the models are never actually retrieved when issuing a mass update.

<a name="mass-assignment"></a>

### Asignación Masiva (Mass Assignment)

También se puede utilizar el método `create` para almacenar un modelo en una única línea. Desde el método se retornará la instancia del modelo insertado. Sin embargo, antes de ello, hay que especificar la propiedad `fillable` o `guarded` del modelo, pues todos los modelos Eloquent poseen protección contra la asignación en masa.

Una vulnerabilidad de asignación masiva tiene lugar cuando un usuario pasa un parámetro HTTP inesperado a través de la solicitud, y este parámetro cambia una columna de la base de datos que no se esperaba. Por ejemplo, un usuario malintencionado podría enviar un parámetro `is_admin` a través de una petición HTTP, el cual se marearía dentro del método `create` del modelo, permitiendo al usuario postularse como un administrador.

Así que, para empezar, hay que definir a qué atributos se les permite la asignación masiva. Esto se establece en la propiedad `fillable` del modelo. Por ejemplo, vamos a permitir la asignación masiva sobre el atributo `name` de nuestro modelo `Flight`:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Flight extends Model
    {
        /**
         * The attributes that are mass assignable.
         *
         * @var array
         */
        protected $fillable = ['name'];
    }
    

Una vez que se han marcado los atributos que permiten la asignación masiva, se puede utilizar el método `create` para insertar un nuevo registro en la base de datos. El método `create` retornará una instancia del modelo guardado:

    $flight = App\Flight::create(['name' => 'Flight 10']);
    

Si ya se tiene una instancia del modelo, se puede usar el método `fill` para completarla con un array de atributos:

    $flight->fill(['name' => 'Flight 22']);
    

#### Guarding Attributes

Mientras que `$fillable` sirve como una "lista blanca" de atributos que pueden ser asignados masivamente, también se puede optar por `$guarded`. La propiedad `guarded` contiene un array de atributos que no pueden ser asignados de forma masiva. El resto de atributos que no se encuentren en el array si podrán. Por lo que, `$guarded` actúa como una "lista negra". Por supuesto, se debe establecer `$fillable` o `$guarded` - nunca ambos. En el ejemplo que sigue, todos los atributos **excepto `price`** contarán con asignación masiva:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Flight extends Model
    {
        /**
         * The attributes that aren't mass assignable.
         *
         * @var array
         */
        protected $guarded = ['price'];
    }
    

Si se desea que todos los atributos se puedan asignar en masa, se puede definir la propiedad `$guarded` como un array vacío:

    /**
     * The attributes that aren't mass assignable.
     *
     * @var array
     */
    protected $guarded = [];
    

<a name="other-creation-methods"></a>

### Otros Metodos de Creación

#### `firstOrCreate`/ `firstOrNew`

Hay otros dos métodos que se pueden utilizar para la creación de modelos asignando atributos masivamente: `firstOrCreate` y `firstOrNew`. El método `firstOrCreate` intentará encontrar un registro en la base de datos utilizando los pares de columna / valor proporcionados. Si el modelo no se puede encontrar en la base de datos, se insertará un registro con los atributos del primer parámetro, junto con los del segundo parámetro opcional.

El método `firstOrNew`, como `firstOrCreate` intentará encontrar un registro en la base de datos que concuerde con los atributos proporcionados. Sin embargo, si no se encuentra, retornará una nueva instancia del modelo. Tener en cuenta que el modelo retornado por `firstOrNew` no se ha almacenado todavía en la base de datos. Hay que llamar al método `save` de forma manual para persistirlo:

    // Retrieve flight by name, or create it if it doesn't exist...
    $flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);
    
    // Retrieve flight by name, or create it with the name and delayed attributes...
    $flight = App\Flight::firstOrCreate(
        ['name' => 'Flight 10'], ['delayed' => 1]
    );
    
    // Retrieve by name, or instantiate...
    $flight = App\Flight::firstOrNew(['name' => 'Flight 10']);
    
    // Retrieve by name, or instantiate with the name and delayed attributes...
    $flight = App\Flight::firstOrNew(
        ['name' => 'Flight 10'], ['delayed' => 1]
    );
    

#### `updateOrCreate`

También se pueden encontrar situaciones en las que desee actualizar un modelo existente o crear un nuevo modelo, si no existe ninguno. Laravel proporciona un método `updateOrCreate` para hacer esto en un solo paso. Al igual que `firstOrCreate`, el método `updateOrCreate` persiste en el modelo, por lo que no es necesario llamar a `save()`:

    // If there's a flight from Oakland to San Diego, set the price to $99.
    // If no matching model exists, create one.
    $flight = App\Flight::updateOrCreate(
        ['departure' => 'Oakland', 'destination' => 'San Diego'],
        ['price' => 99]
    );
    

<a name="deleting-models"></a>

## Eliminar Modelos

Para eliminar un modelo, llamar al método `delete` de la instancia de este:

    $flight = App\Flight::find(1);
    
    $flight->delete();
    

#### Eliminar un Modelo Existente por Clave

En el ejemplo anterior, estamos obteniendo el modelo de la base de datos antes de llamar al método `delete`. Sin embargo, si se conoce la clave primaria del modelo, se puede eliminar sin necesidad de obtenerlo. Para ello, ejecutar el método `destroy`:

    App\Flight::destroy(1);
    
    App\Flight::destroy([1, 2, 3]);
    
    App\Flight::destroy(1, 2, 3);
    

#### Eliminar Modelos de una Consulta

Por supuesto, también se puede ejecutar una declaración de eliminación en un conjunto de modelos. En este ejemplo, se borrarán todos los vuelos marcados como inactivos. Al igual que las actualizaciones masivas, las eliminaciones masivas no activarán ningún evento de modelo para los modelos que se eliminan:

    $deletedRows = App\Flight::where('active', 0)->delete();
    

> {note} Cuando se ejecuta una declaración de eliminación masiva a través de Eloquent, los eventos del modelo `deleting` y `deleted` no se dispararán. Esto se debe a que los modelos nunca se recuperan cuando se ejecuta la eliminación.

<a name="soft-deleting"></a>

### Soft Deleting (Borrado Lógico)

Además de eliminar registros de la base de datos, Eloquent también puede realizar el "borrado lógico" de modelos. Cuando un modelo se borra lógicamente, realmente no se elimina de la base de datos. En su lugar, se establece el atributo `deleted_at` del modelo y se almacena en la base de datos. Si un modelo posee un valor no-null en `deleted_at`, el modelo está soft-deleted. Para habilitar el soft deleting de un modelo, utilizar el trait `Illuminate\Database\Eloquent\SoftDeletes` sobre el modelo y añadir la columna `deleted_at` a la propiedad `$dates`:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\SoftDeletes;
    
    class Flight extends Model
    {
        use SoftDeletes;
    
        /**
         * The attributes that should be mutated to dates.
         *
         * @var array
         */
        protected $dates = ['deleted_at'];
    }
    

Además se debe añadir la columna `deleted_at` a la tabla de la base de datos. El [schema builder](/docs/{{version}}/migrations) de Laravel posee un método para crear esta columna:

    Schema::table('flights', function ($table) {
        $table->softDeletes();
    });
    

Ahora, cuando se ejecute el método `delete` sobre el modelo, la columna `deleted_at` se establecerá con la fecha y hora actuales. Al consultar este modelo, los modelos soft-deleted se excluirán automáticamente de los resultados de la consulta.

Para determinar si una instancia de modelo ha sido soft-deleted, utilizar el método `trashed`:

    if ($flight->trashed()) {
        //
    }
    

<a name="querying-soft-deleted-models"></a>

### Consultar Modelos Soft Deleted

#### Incluir Modelos Soft Deleted

Como se señaló anteriormente, los modelos soft-deleted se excluirán automáticamente de los resultados de consulta. Sin embargo, se puede forzar a incluir estos modelos en un conjunto de resultados utilizando el método `withTrashed` en la consulta:

    $flights = App\Flight::withTrashed()
                    ->where('account_id', 1)
                    ->get();
    

El método `withTrashed` se puede utilizar también en una [relación](/docs/{{version}}/eloquent-relationships):

    $flight->history()->withTrashed()->get();
    

#### Obtener Solo Modelos Soft Deleted

El método `onlyTrashed` obtendrá **únicamente** modelos soft-deleted:

    $flights = App\Flight::onlyTrashed()
                    ->where('airline_id', 1)
                    ->get();
    

#### Restaurar Modelos Soft Deleted

Para "deshacer" el borrado de un modelo soft-deleted a un estado activo, utilizar el método `restore` sobre la instancia del modelo:

    $flight->restore();
    

También se puede utilizar `restore` sobre una consulta para restaurar múltiples modelos rápidamente. Nuevamente, al igual que las otras operaciones "masivas", esto no disparará ningún evento para los modelos que se restauran:

    App\Flight::withTrashed()
            ->where('airline_id', 1)
            ->restore();
    

Al igual que le método `withTrashed`, `restore` se puede utilizar en [relaciones](/docs/{{version}}/eloquent-relationships):

    $flight->history()->restore();
    

#### Eliminar Modelos Permanentemente

Para realmente eliminar un modelo de la base de datos de forma permanente, utilizar el método `forceDelete`:

    // Force deleting a single model instance...
    $flight->forceDelete();
    
    // Force deleting all related models...
    $flight->history()->forceDelete();
    

<a name="query-scopes"></a>

## Query Scopes

<a name="global-scopes"></a>

### Global Scopes

Los Global scopes permiten agregar restricciones a todas las consultas para un modelo determinado. La propia funcionalidad de [soft delete](#soft-deleting) de Laravel utiliza global scopes para extraer únicamente modelos "no eliminados" de la base de datos. Escribir propios global scopes puede proporcionar una manera conveniente y fácil de asegurarse de que cada consulta para un modelo determinado reciba ciertas restricciones.

#### Escribiendo Global Scopes

Escribir un global scope es bastante sencillo. Hay que definir una clase que implemente la interface `Illuminate\Database\Eloquent\Scope`. La interface requiere que se implemente un solo método: `apply`. En el método `apply` se pueden agregar `where` como restricciones a la consulta según sea necesario:

    <?php
    
    namespace App\Scopes;
    
    use Illuminate\Database\Eloquent\Scope;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Builder;
    
    class AgeScope implements Scope
    {
        /**
         * Apply the scope to a given Eloquent query builder.
         *
         * @param  \Illuminate\Database\Eloquent\Builder  $builder
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @return void
         */
        public function apply(Builder $builder, Model $model)
        {
            $builder->where('age', '>', 200);
        }
    }
    

> {tip} Si su ámbito global está agregando columnas a la cláusula de selección de la consulta, debe usar el método `addSelect` en lugar de `select`. Esto evitará el reemplazo involuntario de la query de selección existente de la consulta.

#### Aplicando Global Scopes

Para asignar un global scope a un modelo, se debe sobreescribir el método `boot` de un modelo y usar el método `addGlobalScope`:

    <?php
    
    namespace App;
    
    use App\Scopes\AgeScope;
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * The "booting" method of the model.
         *
         * @return void
         */
        protected static function boot()
        {
            parent::boot();
    
            static::addGlobalScope(new AgeScope);
        }
    }
    

Después de agregar el scope, una consulta a `User::all()` producirá el siguiente SQL:

    select * from `users` where `age` > 200
    

#### Anonymous Global Scopes

Eloquent also allows you to define global scopes using Closures, which is particularly useful for simple scopes that do not warrant a separate class:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Builder;
    
    class User extends Model
    {
        /**
         * The "booting" method of the model.
         *
         * @return void
         */
        protected static function boot()
        {
            parent::boot();
    
            static::addGlobalScope('age', function (Builder $builder) {
                $builder->where('age', '>', 200);
            });
        }
    }
    

#### Removing Global Scopes

If you would like to remove a global scope for a given query, you may use the `withoutGlobalScope` method. The method accepts the class name of the global scope as its only argument:

    User::withoutGlobalScope(AgeScope::class)->get();
    

If you would like to remove several or even all of the global scopes, you may use the `withoutGlobalScopes` method:

    // Remove all of the global scopes...
    User::withoutGlobalScopes()->get();
    
    // Remove some of the global scopes...
    User::withoutGlobalScopes([
        FirstScope::class, SecondScope::class
    ])->get();
    

<a name="local-scopes"></a>

### Local Scopes

Los local scopes permiten definir un conjunto de constraints que se pueden reutilizar fácilmente a lo largo de la aplicación. Por ejemplo, puede que sea frecuente recuperar todos los usuarios considerados "popular". Para definir un scope, simplemente hay que prefijar un modelo Eloquent con `scope`.

Los scopes devolverán siempre una instancia del query buider:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * Scope a query to only include popular users.
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopePopular($query)
        {
            return $query->where('votes', '>', 100);
        }
    
        /**
         * Scope a query to only include active users.
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeActive($query)
        {
            return $query->where('active', 1);
        }
    }
    

#### Utilizar un Local Scope

Una el scope se ha definido, se puede llamar cuando se consulta el modelo. Sin embargo, no es necesario incluir el prefijo `scope` cuando se llama al método. Incluso se pueden encadenar llamadas a varios scopes, por ejemplo:

    $users = App\User::popular()->active()->orderBy('created_at')->get();
    

#### Scopes Dinámicos

A veces es necesario que un scope acepte parámetros. Para ello, simplemente añadir los parámetros adicionales al scope. Scope parameters should be defined after the `$query` parameter:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * Scope a query to only include users of a given type.
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @param mixed $type
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeOfType($query, $type)
        {
            return $query->where('type', $type);
        }
    }
    

Ahora, ya se pueden pasar parámetros al scope:

    $users = App\User::ofType('admin')->get();
    

<a name="events"></a>

## Eventos

Eloquent models fire several events, allowing you to hook into the following points in a model's lifecycle: `retrieved`, `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`. Los eventos permiten ejecutar código de forma sencilla cada vez que una clase de un modelo específico se guarda o actualiza en la base de datos.

The `retrieved` event will fire when an existing model is retrieved from the database. When a new model is saved for the first time, the `creating` and `created` events will fire. Si el modelo ya existe en la base de datos y se llama al método `save`, se lanzarán los eventos `updating` / `updated`. Sin embargo, para ambos casos, se lanzarán los eventos `saving` / `saved`.

To get started, define a `$dispatchesEvents` property on your Eloquent model that maps various points of the Eloquent model's lifecycle to your own [event classes](/docs/{{version}}/events):

    <?php
    
    namespace App;
    
    use App\Events\UserSaved;
    use App\Events\UserDeleted;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    
    class User extends Authenticatable
    {
        use Notifiable;
    
        /**
         * The event map for the model.
         *
         * @var array
         */
        protected $dispatchesEvents = [
            'saved' => UserSaved::class,
            'deleted' => UserDeleted::class,
        ];
    }
    

<a name="observers"></a>

### Observers

If you are listening for many events on a given model, you may use observers to group all of your listeners into a single class. Observers classes have method names which reflect the Eloquent events you wish to listen for. Each of these methods receives the model as their only argument. Laravel does not include a default directory for observers, so you may create any directory you like to house your observer classes:

    <?php
    
    namespace App\Observers;
    
    use App\User;
    
    class UserObserver
    {
        /**
         * Listen to the User created event.
         *
         * @param  \App\User  $user
         * @return void
         */
        public function created(User $user)
        {
            //
        }
    
        /**
         * Listen to the User deleting event.
         *
         * @param  \App\User  $user
         * @return void
         */
        public function deleting(User $user)
        {
            //
        }
    }
    

To register an observer, use the `observe` method on the model you wish to observe. You may register observers in the `boot` method of one of your service providers. In this example, we'll register the observer in the `AppServiceProvider`:

    <?php
    
    namespace App\Providers;
    
    use App\User;
    use App\Observers\UserObserver;
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
            User::observe(UserObserver::class);
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