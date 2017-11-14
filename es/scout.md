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
    

Finalmente, agregue el *trait* `Laravel\Scout\Searchable` al modelo en el que desea buscar. This trait will register a model observer to keep the model in sync with your search driver:

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

If you would like to add a collection of models to your search index via an Eloquent query, you may chain the `searchable` method onto an Eloquent query. The `searchable` method will [chunk the results](/docs/{{version}}/eloquent#chunking-results) of the query and add the records to your search index. Again, if you have configured Scout to use queues, all of the chunks will be added in the background by your queue workers:

    // Adding via Eloquent query...
    App\Order::where('price', '>', 100)->searchable();
    
    // You may also add records via relationships...
    $user->orders()->searchable();
    
    // You may also add records via collections...
    $orders->searchable();
    

The `searchable` method can be considered an "upsert" operation. In other words, if the model record is already in your index, it will be updated. If it does not exist in the search index, it will be added to the index.

<a name="updating-records"></a>

### Updating Records

To update a searchable model, you only need to update the model instance's properties and `save` the model to your database. Scout will automatically persist the changes to your search index:

    $order = App\Order::find(1);
    
    // Update the order...
    
    $order->save();
    

You may also use the `searchable` method on an Eloquent query to update a collection of models. If the models do not exist in your search index, they will be created:

    // Updating via Eloquent query...
    App\Order::where('price', '>', 100)->searchable();
    
    // You may also update via relationships...
    $user->orders()->searchable();
    
    // You may also update via collections...
    $orders->searchable();
    

<a name="removing-records"></a>

### Removing Records

To remove a record from your index, simply `delete` the model from the database. This form of removal is even compatible with [soft deleted](/docs/{{version}}/eloquent#soft-deleting) models:

    $order = App\Order::find(1);
    
    $order->delete();
    

If you do not want to retrieve the model before deleting the record, you may use the `unsearchable` method on an Eloquent query instance or collection:

    // Removing via Eloquent query...
    App\Order::where('price', '>', 100)->unsearchable();
    
    // You may also remove via relationships...
    $user->orders()->unsearchable();
    
    // You may also remove via collections...
    $orders->unsearchable();
    

<a name="pausing-indexing"></a>

### Pausing Indexing

Sometimes you may need to perform a batch of Eloquent operations on a model without syncing the model data to your search index. You may do this using the `withoutSyncingToSearch` method. This method accepts a single callback which will be immediately executed. Any model operations that occur within the callback will not be synced to the model's index:

    App\Order::withoutSyncingToSearch(function () {
        // Perform model actions...
    });
    

<a name="searching"></a>

## Searching

You may begin searching a model using the `search` method. The search method accepts a single string that will be used to search your models. You should then chain the `get` method onto the search query to retrieve the Eloquent models that match the given search query:

    $orders = App\Order::search('Star Trek')->get();
    

Since Scout searches return a collection of Eloquent models, you may even return the results directly from a route or controller and they will automatically be converted to JSON:

    use Illuminate\Http\Request;
    
    Route::get('/search', function (Request $request) {
        return App\Order::search($request->search)->get();
    });
    

If you would like to get the raw results before they are converted to Eloquent models, you should use the `raw` method:

    $orders = App\Order::search('Star Trek')->raw();
    

Search queries will typically be performed on the index specified by the model's [`searchableAs`](#configuring-model-indexes) method. However, you may use the `within` method to specify a custom index that should be searched instead:

    $orders = App\Order::search('Star Trek')
        ->within('tv_shows_popularity_desc')
        ->get();
    

<a name="where-clauses"></a>

### Cláusulas Where

Scout allows you to add simple "where" clauses to your search queries. Currently, these clauses only support basic numeric equality checks, and are primarily useful for scoping search queries by a tenant ID. Since a search index is not a relational database, more advanced "where" clauses are not currently supported:

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