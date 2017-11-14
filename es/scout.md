# Laravel Scout

- [Introducción](#introduction)
- [Instalación](#installation) 
    - [Colas – *Queueing*](#queueing)
    - [Prerequisitos del driver](#driver-prerequisites)
- [Configuración](#configuration) 
    - [Configurar el indexado de modelos](#configuring-model-indexes)
    - [Configurar datos para la búsqueda](#configuring-searchable-data)
- [Indexar](#indexing) 
    - [Importación por lotes](#batch-import)
    - [Agregar registros](#adding-records)
    - [Actualizar registros](#updating-records)
    - [Eliminar registros](#removing-records)
    - [Pausar el indexado](#pausing-indexing)
- [Buscar](#searching) 
    - [Cláusulas *where*](#where-clauses)
    - [Paginación](#pagination)
- [Motores propios](#custom-engines)

<a name="introduction"></a>

## Introducción

Laravel Scout proporciona una solución sencilla, basada en *drivers* para añadir la búsqueda de texto completo a sus modelos [Eloquent](/docs/{{version}}/eloquent). Usando los observadores de modelo, Scout automáticamente mantendrá sus índices de búsqueda sincronizados con sus registros Eloquent.

Actualmente, Scout trabaja con un *drivers* [Algolia](https://www.algolia.com/); sin embargo, es bastante sencillo extender Scout con sus propias implementaciones de búsqueda.

<a name="installation"></a>

## Instalación

Primero, instale Scout utilizando Composer:

    composer require laravel/scout
    

Después de instalar Scout, debe publicar la configuración usando el comando Artisan `vendor:publish`. Este comando publicará el archivo de configuración `scout.php` en su directorio `config`:

    php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"
    

Finalmente, agregue el *trait* `Laravel\Scout\Searchable` al modelo en el que desea buscar. Este *trait* registrará un observador de modelo para mantenerlo sincronizado con su *drivers* de búsqueda:

    <?php
    
    namespace App;
    
    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;
    
    class Post extends Model
    {
        use Searchable;
    }
    

<a name="queueing"></a>

### Colas – *Queueing*

Aunque no es estrictamente necesario para usar Scout, considere seriamente configurar un [queue driver](/docs/{{version}}/queues) antes de usar la librería. La ejecución de un *queue driver* le permitirá a Scout poner en cola a todas las operaciones que sincronicen la información de su modelo con sus índices de búsqueda, proporcionando tiempos de respuesta mucho mejores para la interfaz web de su aplicación.

Una vez que haya configurado un `queue driver`, ajuste el valor de la opción `queue` en su archivo de configuración `config/scout.php` a <0>true</0>:

    'queue' => true,
    

<a name="driver-prerequisites"></a>

### Prerequisitos del driver

#### Algolia

Cuando utilice el *driver* Algolia, debe configurar sus credenciales Algolia `id` y `secret` en su fichero de configuración `config/scout.php`. Una vez que sus credenciales hayan sido configuradas, también necesitará instalar el SDK PHP de Algolia a través de Composer:

    composer require algolia/algoliasearch-client-php
    

<a name="configuration"></a>

## Configuración

<a name="configuring-model-indexes"></a>

### Configurar el indexado de modelos

Cada modelo Eloquent se sincroniza con un determinado "index" de búsqueda, que contiene todos los registros en donde se puede buscar para ese modelo. En otras palabras, puede pensar en cada índice como una tabla MySQL. De forma predeterminada, cada modelo persistirá en un índice que normalmente coincide con el nombre de la "tabla" del modelo. Normalmente, esta es la forma plural del nombre del modelo; sin embargo, se es libre de personalizar el índice anulando el método `buscableAs` en el modelo:

    <?php
    
    namespace App;
    
    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;
    
    class Post extends Model
    {
        use Searchable;
    
        /**
         * Get the index name for the model.
         *
         * @return string
         */
        public function searchableAs()
        {
            return 'posts_index';
        }
    }
    

<a name="configuring-searchable-data"></a>

### Configurar datos para la búsqueda

Por defecto, la forma completa del método `toArray` de un modelo se persistirá en su índice de búsqueda. Si desea personalizar los datos que se sincronizan con el índice de búsqueda, puede sustituir el método `toSearchableArray` en el modelo:

    <?php
    
    namespace App;
    
    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;
    
    class Post extends Model
    {
        use Searchable;
    
        /**
         * Get the indexable data array for the model.
         *
         * @return array
         */
        public function toSearchableArray()
        {
            $array = $this->toArray();
    
            // Customize array...
    
            return $array;
        }
    }
    

<a name="indexing"></a>

## Indexar

<a name="batch-import"></a>

### Importación por lotes

Si está instalando Scout en un proyecto existente, es posible que ya tenga registros de base de datos que necesite importar a su controlador de búsqueda. Scout proporciona un comando Artisan `import` que puede utilizar para importar todos los registros existentes en sus índices de búsqueda:

    php artisan scout:import "App\Post"
    

<a name="adding-records"></a>

### Agregar registros

Una vez que haya agregado el *trait* `Laravel\Scout\Searchable` a un modelo, todo lo que necesita hacer es `save` en la instancia del modelo y automáticamente se agregará a su índice de búsqueda. Si usted ha configurado Scout para [use queues](#queueing) esta operación será realizada en segundo plano por su *queue worker*:

    $order = new App\Order;
    
    // ...
    
    $order->save();
    

#### Agregar vía query

Si desea agregar una colección de modelos a su índice de búsqueda mediante una consulta Eloquent, puede encadenar el método `searchable` a la consulta. El método `searchable` [fragmentara los resultados](/docs/{{version}}/eloquent#chunking-results) de la consulta y añadirá los registros a su índice de búsqueda. Una vez más, si ha configurado Scout para usar colas, todos los "fragmentos" se añadirán en segundo plano:

    // Adding via Eloquent query...
    App\Order::where('price', '>', 100)->searchable();
    
    // You may also add records via relationships...
    $user->orders()->searchable();
    
    // You may also add records via collections...
    $orders->searchable();
    

El método `searchable` puede considerarse una operación "upsert". En otras palabras, si el registro del modelo ya está en su índice, se actualizará. Si no existe en el índice de búsqueda, se añadirá.

<a name="updating-records"></a>

### Actualizar registros

Para actualizar un modelo *searchable*, sólo necesita actualizar las propiedades de la instancia del modelo y `guardarlo (save)` en su base de datos. Scout persistirá automáticamente los cambios en su índice de búsqueda:

    $order = App\Order::find(1);
    
    // Update the order...
    
    $order->save();
    

También puede utilizar el método `searchable` en una consulta Eloquent para actualizar una colección de modelos. Si los modelos no existen en su índice de búsqueda, se crearán:

    // Updating via Eloquent query...
    App\Order::where('price', '>', 100)->searchable();
    
    // You may also update via relationships...
    $user->orders()->searchable();
    
    // You may also update via collections...
    $orders->searchable();
    

<a name="removing-records"></a>

### Eliminar registros

Para eliminar un registro de su índice, simplemente `elimine (delete)` el modelo de la base de datos. Esta forma de eliminación es incluso compatible con los modelos [soft deleted](/docs/{{version}}/eloquent#soft-deleting):

    $order = App\Order::find(1);
    
    $order->delete();
    

Si no desea recuperar el modelo antes de borrar el registro, puede utilizar el método `unsearchable` en una instancia o colección de consulta Eloquent:

    // Removing via Eloquent query...
    App\Order::where('price', '>', 100)->unsearchable();
    
    // You may also remove via relationships...
    $user->orders()->unsearchable();
    
    // You may also remove via collections...
    $orders->unsearchable();
    

<a name="pausing-indexing"></a>

### Pausar el indexado

A veces puede ser necesario realizar un lote de operaciones Eloquent en un modelo sin sincronizar los datos con el índice de búsqueda. Puede hacerlo utilizando el método `withoutSyncingToSearch`. Este método acepta una única llamada de retorno que se ejecutará inmediatamente. Cualquier operación del modelo que ocurra dentro de la llamada de retorno no se sincronizará con el índice del modelo:

    App\Order::withoutSyncingToSearch(function () {
        // Perform model actions...
    });
    

<a name="searching"></a>

## Buscar

Puede comenzar a buscar un modelo utilizando el método `search`. El método de búsqueda acepta una sola cadena que se utilizará para buscar en sus modelos. A continuación, debería encadenar el método `get` en la consulta de búsqueda para recuperar los modelos Eloquent que coinciden:

    $orders = App\Order::search('Star Trek')->get();
    

Dado que las búsquedas Scout devuelven una colección de modelos Eloquent, incluso puede devolver los resultados directamente desde una ruta o controlador y se convertirán automáticamente a JSON:

    use Illuminate\Http\Request;
    
    Route::get('/search', function (Request $request) {
        return App\Order::search($request->search)->get();
    });
    

Si desea obtener los resultados brutos antes de convertirlos a modelos Eloquent, debe utilizar el método `raw`:

    $orders = App\Order::search('Star Trek')->raw();
    

Las consultas de búsqueda normalmente se realizarán en el índice especificado por el método [`searchableAs`](#configuring-model-indexes) del modelo. Sin embargo, puede utilizar el método `within` para especificar un índice personalizado:

    $orders = App\Order::search('Star Trek')
        ->within('tv_shows_popularity_desc')
        ->get();
    

<a name="where-clauses"></a>

### Cláusulas Where

Scout le permite añadir cláusulas simples "where" a sus consultas de búsqueda. Actualmente, estas cláusulas sólo admiten comprobaciones básicas de igualdad numérica, y son útiles principalmente para determinar el alcance de las consultas de búsqueda por parte de un *tenant ID*. Dado que una búsqueda por indice no es relacional, las cláusulas "where" más avanzadas no son actualmente compatibles:

    $orders = App\Order::search('Star Trek')->where('user_id', 1)->get();
    

<a name="pagination"></a>

### Paginación

Además de recuperar una colección de modelos, puede paginar los resultados de su búsqueda utilizando el método `paginate`. Este método devolverá una instancia `Paginator` igual que si hubiera [paginado una consulta Eloquent tradicional](/docs/{{version}}/pagination):

    $orders = App\Order::search('Star Trek')->paginate();
    

Puede especificar cuántos modelos desea recuperar por página pasando la cantidad como primer argumento al método `paginate`:

    $orders = App\Order::search('Star Trek')->paginate(15);
    

Una vez que se hayan recuperado los resultados, puede mostrarlos y colocar los enlaces de la página usando [Blade](/docs/{{version}}/blade) como si hubiera paginado una consulta Eloquent tradicional:

    <div class="container">
        @foreach ($orders as $order)
            {{ $order->price }}
        @endforeach
    </div>
    
    {{ $orders->links() }}
    

<a name="custom-engines"></a>

## Motores propios

#### Escribiendo el motor

Si los buscadores Scout incorporados no se ajustan a sus necesidades, puede escribir su propio motor de búsqueda personalizado y registrarlo con Scout. Su motor debe extender la clase abstracta `Laravel\Scout\Engines\Engine`. Esta clase abstracta contiene cinco métodos que su motor debe implementar:

    use Laravel\Scout\Builder;
    
    abstract public function update($models);
    abstract public function delete($models);
    abstract public function search(Builder $builder);
    abstract public function paginate(Builder $builder, $perPage, $page);
    abstract public function map($results, $model);
    

Puede ser útil revisar las implementaciones de estos métodos en la clase `Laravel\Scout\Engines\AlgoliaEngine`. Esta clase le proporcionará un buen punto de partida para aprender a implementar cada uno de estos métodos en su propio motor.

#### Registrando el motor

Una vez escrito el motor personalizado, puede registrarse con Scout usando el método `extend` del gestor del motor de Scout. Debe llamar al método `extend` desde el método `boot` de su `AppServiceProvider` o desde cualquier otro proveedor de servicios utilizado por su aplicación. Por ejemplo, si ha escrito un `MySqlSearchEngine`, puede registrarlo así:

    use Laravel\Scout\EngineManager;
    
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        resolve(EngineManager::class)->extend('mysql', function () {
            return new MySqlSearchEngine;
        });
    }
    

Una vez que su motor ha sido registrado, puede especificarlo como `driver` Scout predeterminado en su archivo de configuración `config/scout.php`:

    'driver' => 'mysql',