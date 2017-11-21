# Database: Paginación

- [Introducción](#introduction)
- [Uso básico](#basic-usage) 
    - [Paginar resultados del **query builder**](#paginating-query-builder-results)
    - [Paginar resultados de Eloquent](#paginating-eloquent-results)
    - [Crear un *paginador* manualmente](#manually-creating-a-paginator)
- [Mostrar los resultados de la paginación](#displaying-pagination-results) 
    - [Convertir resultados a JSON](#converting-results-to-json)
- [Personalizar la vista de la paginación](#customizing-the-pagination-view)
- [Métodos de la instancia de paginación](#paginator-instance-methods)

<a name="introduction"></a>

## Introducción

En otros frameworks, la paginación puede ser muy difícil. El *paginador* de Laravel está integrado con [*query builder*](/docs/{{version}}/queries) y [Eloquent ORM](/docs/{{version}}/eloquent), proporcionando una paginación conveniente y fácil de usar de los resultados de base de datos. El HTML generado por el *paginador* es compatible con el framework [Bootstrap CSS](https://getbootstrap.com/).

<a name="basic-usage"></a>

## Uso básico

<a name="paginating-query-builder-results"></a>

### Paginar resultados del **query builder**

Hay varias formas para paginar elementos. La más simple es mediante el método `paginate` en el [*query builder*](/docs/{{version}}/queries) o una [consulta de Eloquent](/docs/{{version}}/eloquent). El método `paginate` se encarga automáticamente de fijar el límite y el *offset* adecuados en función de la página actual que esté viendo el usuario. De forma predeterminada, la página actual se detecta por el valor del argumento `page` que es una cadena de consulta en la petición HTTP. Por supuesto, este valor es automáticamente detectado por Laravel y se inserta automáticamente en los enlaces generados por el *paginador*.

En este ejemplo, el único argumento que se pasa al método `paginate` es el número de elementos que desea que se muestren "por página". En este caso, se especifica que se quieren mostrar `15` elementos por página:

    <?php
    
    namespace App\Http\Controllers;
    
    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;
    
    class UserController extends Controller
    {
        /**
         * Show all of the users for the application.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->paginate(15);
    
            return view('user.index', ['users' => $users]);
        }
    }
    

> {note} Actualmente, las operaciones de paginación que utilizan una sentencia `groupBy` pueden ser ejecutadas por Laravel de forma eficiente. Si se necesita utilizar `groupBy` con un conjunto de resultados paginado, es aconsejable realizar la consulta a la base de datos y crear un *paginador* manual.

#### "Paginación simple"

Si sólo necesita mostrar los enlaces simples "Siguiente" y "Anterior" en su vista de paginación, puede utilizar el método `simplePaginate` para realizar una consulta más eficiente. Esto es muy útil para grandes conjuntos de datos cuando no necesita mostrar un enlace para cada número de página al renderizar su vista:

    $users = DB::table('users')->simplePaginate(15);
    

<a name="paginating-eloquent-results"></a>

### Paginar resultados de Eloquent

También se pueden paginar consultas de [Eloquent](/docs/{{version}}/eloquent). En este ejemplo, se paginará el modelo `User` con `15` elementos por página. Como se puede ver, la sintaxis es casi idéntica a la paginación de resultados con :

    $users = App\User::paginate(15);
    

Por supuesto, se puede llamar a `paginate` después de establecer otras limitaciones en la consulta, tales como cláusulas `where`:

    $users = User::where('votes', '>', 100)->paginate(15);
    

También se puede utilizar el método `simplePaginate` cuando se paginan modelos de Eloquent:

    $users = User::where('votes', '>', 100)->simplePaginate(15);
    

<a name="manually-creating-a-paginator"></a>

### Crear un *paginador* manualmente

Se puede crear una paginación manualmente, para esto hay que pasar un array de elementos. Se puede hacer mediante la creación de una instancia de `Illuminate\Pagination\Paginator` o `Illuminate\Pagination\LengthAwarePaginator`, dependiendo de las necesidades.

La clase `Paginator` no necesita saber el número total de elementos del conjunto de resultados; sin embargo, debido a esto, no tiene métodos para recuperar el índice de la última página. El `LengthAwarePaginator` acepta casi los mismos parámetros que `Paginator`; sin embargo, requiere contar el número total de elementos del conjunto de resultados.

En otras palabras, `Paginator` corresponde al método `simplePaginate` en Eloquent, mientras que `LengthAwarePaginator` corresponde al método `paginate`.

> {note} Al crear manualmente una instancia de paginator, se debe "acortar" de forma manual el *array* de resultados que se pasa a paginator. Si no se está seguro sobre cómo hacerlo, se puede revisar la función [array_slice](https://secure.php.net/manual/en/function.array-slice.php) de PHP.

<a name="displaying-pagination-results"></a>

## Mostrar los resultados de la paginación

Al llamar al método `paginate`, se recibirá una instancia de `Illuminate\Pagination\LengthAwarePaginator`. Al llamar al método `simplePaginate`, se recibirá una instancia de `Illuminate\Pagination\Paginator`. Estos objetos proporcionan varios métodos que describen el conjunto de resultados. Además de estos métodos de ayuda, las instancias de *paginator* son iterables y pueden ser recorridas como un *array*. Por lo tanto, una vez que se han obtenido los resultados, se pueden mostrar y crear los enlaces de las páginas usando [Blade](/docs/{{version}}/blade):

    <div class="container">
        @foreach ($users as $user)
            {{ $user->name }}
        @endforeach
    </div>
    
    {{ $users->links() }}
    

El método `links` mostrará los enlaces al resto de las páginas del conjunto de resultados. Cada uno de estos enlaces ya contendrá la variable apropiada `page` de la cadena de consulta. Recuerde, el HTML generado por el método `links` es compatible con el [Bootstrap CSS framework](https://getbootstrap.com).

#### Personalizar la URI del *paginador*

El método `withPath` permite personalizar la URI utilizada por el *paginador* al generar enlaces. Por ejemplo, si desea que el *paginador* genere enlaces como `http://example.com/custom/url?page=N`, debe pasar `custom/url` al método `withPath`:

    Route::get('users', function () {
        $users = App\User::paginate(15);
    
        $users->withPath('custom/url');
    
        //
    });
    

#### Añadir variables a los links de paginación

Se puede agregar información a la cadena de consulta de los enlaces de la paginación mediante el método `appends`. Por ejemplo, para añadir `sort=votes` a cada enlace de la paginación, se debe hacer la siguiente llamada a `appends`:

    {{ $users->appends(['sort' => 'votes'])->links() }}
    

Si se desea agregar un ancla a las direcciones URL del *paginador*, se puede utilizar el método `fragment`. Por ejemplo, para agregar `#foo` al final de cada enlace de la paginación, se debe realizar la siguiente llamada al método `fragment`:

    {{ $users->fragment('foo')->links() }}
    

<a name="converting-results-to-json"></a>

### Convertir resultados a JSON

Las clases de resultados del *paginador* de Laravel implementan la interfaz `Illuminate\Contracts\Support\Jsonable` y exponen el método `toJson`, por lo que es muy fácil convertir los resultados de paginación a JSON. También se puede convertir una instancia *paginator* a JSON simplemente retornándola desde una ruta o acción de controlador:

    Route::get('users', function () {
        return App\User::paginate();
    });
    

El JSON del *paginator* incluirá información como `total`, `current_page`, `last_page` y más. Los objetos del resultado actual estarán disponibles a través de la clave `data` en el *array* JSON. Este es un ejemplo de JSON generado al retornar una instancia de paginator en una ruta:

    {
       "total": 50,
       "per_page": 15,
       "current_page": 1,
       "last_page": 4,
       "first_page_url": "http://laravel.app?page=1",
       "last_page_url": "http://laravel.app?page=4",
       "next_page_url": "http://laravel.app?page=2",
       "prev_page_url": null,
       "path": "http://laravel.app",
       "from": 1,
       "to": 15,
       "data":[
            {
                // Result Object
            },
            {
                // Result Object
            }
       ]
    }
    

<a name="customizing-the-pagination-view"></a>

## Personalizar la vista de la paginación

Por defecto, las vistas usadas para mostrar los enlaces de paginación son compatibles con el framework Bootstrap CSS. Sin embargo, si no está usando Bootstrap, es libre de definir sus propias vistas para renderizar estos enlaces. Cuando llame al método `links` en una instancia de *paginator*, pase el nombre de la vista como primer argumento al método:

    {{ $paginator->links('view.name') }}
    
    // Passing data to the view...
    {{ $paginator->links('view.name', ['foo' => 'bar']) }}
    

Sin embargo, la forma más fácil de personalizar las vistas de paginación es exportarlas a su directorio `resources/views/vendor` utilizando el comando Artisan `vendor:publish`:

    php artisan vendor:publish --tag=laravel-pagination
    

Este comando colocará las vistas en el directorio `resources/views/vendor/pagination`. El archivo `default.blade.php` dentro de este directorio corresponde a la vista de paginación predeterminada. Simplemente edite este archivo para modificar el HTML de paginación.

<a name="paginator-instance-methods"></a>

## Métodos de la instancia de paginación

Cada instancia de *paginator* proporciona información adicional de paginación a través de los siguientes métodos:

- `$results->count()`
- `$results->currentPage()`
- `$results->firstItem()`
- `$results->hasMorePages()`
- `$results->lastItem()`
- `$results->lastPage() (Not available when using simplePaginate)`
- `$results->nextPageUrl()`
- `$results->perPage()`
- `$results->previousPageUrl()`
- `$results->total() (Not available when using simplePaginate)`
- `$results->url($page)`