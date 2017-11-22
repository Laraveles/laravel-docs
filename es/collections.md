# Colecciones

- [Introducción](#introduction) 
    - [Crear colecciones](#creating-collections)
    - [Extender colecciones](#extending-collections)
- [Métodos disponibles](#available-methods)
- [Nivel superior de mensajes](#higher-order-messages)

<a name="introduction"></a>

## Introducción

La clase `Illuminate\Support\Collection` proporciona un encapsulador muy fluido para trabajar con *arrays* de datos. Por ejemplo, revisar el siguiente código. Se utiliza el *helper* `collect` para crear una nueva instancia de collection desde un *array*, ejecutar la función `strtoupper` sobre cada elemento y eliminar cualquier elemento vacío:

    $collection = collect(['taylor', 'abigail', null])->map(function ($name) {
        return strtoupper($name);
    })
    ->reject(function ($name) {
        return empty($name);
    });
    

Como se puede apreciar, la clase `Collection` permite encadenar estos métodos para llevar a cabo mapeos y reducción del *array* de datos subyacente a la colección. En general, las colecciones son inmutables, lo que quiere decir que cada método de `Collection` retorna una instancia completamente nueva de `Collection`.

<a name="creating-collections"></a>

### Crear colecciones

Como ya se ha mencionado, el *helper* `collect` retorna una nueva instancia de `Illuminate\Support\Collection` para el *array* dado. Por lo tanto, crear una colección es tan simple como:

    $collection = collect([1, 2, 3]);
    

> {tip} Los resultados de consultas [Eloquent](/docs/{{version}}/eloquent) se devuelven siempre como instancias de `Collection`.

<a name="extending-collections"></a>

### Extender colecciones

Las colecciones son "macroables", por lo que permiten añadir métodos adicionales a la clase `Collection` en tiempo de ejecución. Por ejemplo, el siguiente código añade el método `toUpper` a la clase `Collection`:

    use Illuminate\Support\Str;
    
    Collection::macro('toUpper', function () {
        return $this->map(function ($value) {
            return Str::upper($value);
        });
    });
    
    $collection = collect(['first', 'second']);
    
    $upper = $collection->toUpper();
    
    // ['FIRST', 'SECOND']
    

Normalmente se deben declarar las *macros* en un [service provider](/docs/{{version}}/providers).

<a name="available-methods"></a>

## Métodos disponibles

Para el resto de este documento, se describirá cada uno de los métodos disponibles para la clase `Collection`. Recordar, todos estos métodos se pueden encadenar para manipular el *array* subyacente. Además, en prácticamente cada método se retorna una instancia de `Collection`, permitiendo preservar la copia original de la colección cuando sea necesario:

<style>
    #collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    #collection-method-list a {
        display: block;
    }
</style>

<div id="collection-method-list">
  <p>
    <a href="#method-all">all</a> <a href="#method-average">average</a> <a href="#method-avg">avg</a> <a href="#method-chunk">chunk</a> <a href="#method-collapse">collapse</a> <a href="#method-combine">combine</a> <a href="#method-concat">concat</a> <a href="#method-contains">contains</a> <a href="#method-containsstrict">containsStrict</a> <a href="#method-count">count</a> <a href="#method-crossjoin">crossJoin</a> <a href="#method-dd">dd</a> <a href="#method-diff">diff</a> <a href="#method-diffassoc">diffAssoc</a> <a href="#method-diffkeys">diffKeys</a> <a href="#method-dump">dump</a> <a href="#method-each">each</a> <a href="#method-eachspread">eachSpread</a> <a href="#method-every">every</a> <a href="#method-except">except</a> <a href="#method-filter">filter</a> <a href="#method-first">first</a> <a href="#method-flatmap">flatMap</a> <a href="#method-flatten">flatten</a> <a href="#method-flip">flip</a> <a href="#method-forget">forget</a> <a href="#method-forpage">forPage</a> <a href="#method-get">get</a> <a href="#method-groupby">groupBy</a> <a href="#method-has">has</a> <a href="#method-implode">implode</a> <a href="#method-intersect">intersect</a> <a href="#method-intersectbykeys">intersectByKeys</a> <a href="#method-isempty">isEmpty</a> <a href="#method-isnotempty">isNotEmpty</a> <a href="#method-keyby">keyBy</a> <a href="#method-keys">keys</a> <a href="#method-last">last</a> <a href="#method-macro">macro</a> <a href="#method-make">make</a> <a href="#method-map">map</a> <a href="#method-mapinto">mapInto</a> <a href="#method-mapspread">mapSpread</a> <a href="#method-maptogroups">mapToGroups</a> <a href="#method-mapwithkeys">mapWithKeys</a> <a href="#method-max">max</a> <a href="#method-median">median</a> <a href="#method-merge">merge</a> <a href="#method-min">min</a> <a href="#method-mode">mode</a> <a href="#method-nth">nth</a> <a href="#method-only">only</a> <a href="#method-pad">pad</a> <a href="#method-partition">partition</a> <a href="#method-pipe">pipe</a> <a href="#method-pluck">pluck</a> <a href="#method-pop">pop</a> <a href="#method-prepend">prepend</a> <a href="#method-pull">pull</a> <a href="#method-push">push</a> <a href="#method-put">put</a> <a href="#method-random">random</a> <a href="#method-reduce">reduce</a> <a href="#method-reject">reject</a> <a href="#method-reverse">reverse</a> <a href="#method-search">search</a> <a href="#method-shift">shift</a> <a href="#method-shuffle">shuffle</a> <a href="#method-slice">slice</a> <a href="#method-sort">sort</a> <a href="#method-sortby">sortBy</a> <a href="#method-sortbydesc">sortByDesc</a> <a href="#method-splice">splice</a> <a href="#method-split">split</a> <a href="#method-sum">sum</a> <a href="#method-take">take</a> <a href="#method-tap">tap</a> <a href="#method-times">times</a> <a href="#method-toarray">toArray</a> <a href="#method-tojson">toJson</a> <a href="#method-transform">transform</a> <a href="#method-union">union</a> <a href="#method-unique">unique</a> <a href="#method-uniquestrict">uniqueStrict</a> <a href="#method-unless">unless</a> <a href="#method-unwrap">unwrap</a> <a href="#method-values">values</a> <a href="#method-when">when</a> <a href="#method-where">where</a> <a href="#method-wherestrict">whereStrict</a> <a href="#method-wherein">whereIn</a> <a href="#method-whereinstrict">whereInStrict</a> <a href="#method-wherenotin">whereNotIn</a> <a href="#method-wherenotinstrict">whereNotInStrict</a> <a href="#method-wrap">wrap</a> <a href="#method-zip">zip</a>
  </p>
</div>

<a name="method-listing"></a>

## Lista de métodos

<style>
    #collection-method code {
        font-size: 14px;
    }

    #collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="method-all"></a>

#### `all()` {#collection-method.first-collection-method}

El método `all` retorna el array subyacente que representa la colección:

    collect([1, 2, 3])->all();
    
    // [1, 2, 3]
    

<a name="method-average"></a>

#### `average()` {#collection-method}

Alias para el método [`avg`](#method-avg).

<a name="method-avg"></a>

#### `avg()` {#collection-method}

El método `avg` retorna la [media aritmética](https://en.wikipedia.org/wiki/Average) de una clave concreta:

    $average = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->avg('foo');
    
    // 20
    
    $average = collect([1, 1, 2, 4])->avg();
    
    // 2
    

<a name="method-chunk"></a>

#### `chunk()` {#collection-method}

El método `chunk` divide la colección en varias colecciones más pequeñas del tamaño especificado:

    $collection = collect([1, 2, 3, 4, 5, 6, 7]);
    
    $chunks = $collection->chunk(4);
    
    $chunks->toArray();
    
    // [[1, 2, 3, 4], [5, 6, 7]]
    

Este método es especialmente útil en [vistas](/docs/{{version}}/views) cuando se trabaja con un sistema de *grid* como [Bootstrap](https://getbootstrap.com/css/#grid). Imaginar que se tiene una colección de modelos [Eloquent](/docs/{{version}}/eloquent) y se quiere mostrar en un grid:

    @foreach ($products->chunk(3) as $chunk)
        <div class="row">
            @foreach ($chunk as $product)
                <div class="col-xs-4">{{ $product->name }}</div>
            @endforeach
        </div>
    @endforeach
    

<a name="method-collapse"></a>

#### `collapse()` {#collection-method}

El método `collapse` colapsa una colección de *arrays* en una colección simple y plana:

    $collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);
    
    $collapsed = $collection->collapse();
    
    $collapsed->all();
    
    // [1, 2, 3, 4, 5, 6, 7, 8, 9]
    

<a name="method-combine"></a>

#### `combine()` {#collection-method}

El método `combine` combina las claves de la colección con valores de otro *array* o colección:

    $collection = collect(['name', 'age']);
    
    $combined = $collection->combine(['George', 29]);
    
    $combined->all();
    
    // ['name' => 'George', 'age' => 29]
    

<a name="method-concat"></a>

#### `concat()` {#collection-method}

El método `concat` anexa los valores del `array` o colección dados al final de la colección:

    $collection = collect(['John Doe']);
    
    $concatenated = $collection->concat(['Jane Doe'])->concat(['name' => 'Johnny Doe']);
    
    $concatenated->all();
    
    // ['John Doe', 'Jane Doe', 'Johnny Doe']
    

<a name="method-contains"></a>

#### `contains()` {#collection-method}

El método `contains` determina si la colección contiene un elemento concreto:

    $collection = collect(['name' => 'Desk', 'price' => 100]);
    
    $collection->contains('Desk');
    
    // true
    
    $collection->contains('New York');
    
    // false
    

También se puede pasar un par de clave / valor al método `contains`, el cual determinará si el par existe en la colección:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);
    
    $collection->contains('product', 'Bookcase');
    
    // false
    

Finalmente, se puede pasar un *callback* al método `contains` para realizar un test propio:

    $collection = collect([1, 2, 3, 4, 5]);
    
    $collection->contains(function ($value, $key) {
        return $value > 5;
    });
    
    // false
    

El método `contains` utiliza comparaciones "débiles" al comprobar los valores de los elementos, por lo que una cadena con un valor entero se considerará igual a un entero del mismo valor. Utilizar el método [`containsStrict`](#method-containsstrict) para filtrar utilizando comparaciones "estrictas".

<a name="method-containsstrict"></a>

#### `containsStrict()` {#collection-method}

Este método tiene la misma firma que el método [`contains`](#method-contains); sin embargo, todos los valores se comparan utilizando operadores "estrictos".

<a name="method-count"></a>

#### `count()` {#collection-method}

El método `count` retorna el total de elementos en la colección:

    $collection = collect([1, 2, 3, 4]);
    
    $collection->count();
    
    // 4
    

<a name="method-crossjoin"></a>

#### `crossJoin()` {#collection-method}

El método `crossJoin` mezclará los valores de la colección con el *array* o colección dado, retornando un producto cartesiano con todas las permutaciones posibles:

    $collection = collect([1, 2]);
    
    $matrix = $collection->crossJoin(['a', 'b']);
    
    $matrix->all();
    
    /*
        [
            [1, 'a'],
            [1, 'b'],
            [2, 'a'],
            [2, 'b'],
        ]
    */
    
    $collection = collect([1, 2]);
    
    $matrix = $collection->crossJoin(['a', 'b'], ['I', 'II']);
    
    $matrix->all();
    
    /*
        [
            [1, 'a', 'I'],
            [1, 'a', 'II'],
            [1, 'b', 'I'],
            [1, 'b', 'II'],
            [2, 'a', 'I'],
            [2, 'a', 'II'],
            [2, 'b', 'I'],
            [2, 'b', 'II'],
        ]
    */
    

<a name="method-dd"></a>

#### `dd()` {#collection-method}

El método `dd` vuelca el contenido de la colección y finaliza con la ejecución del script:

    $collection = collect(['John Doe', 'Jane Doe']);
    
    $collection->dd();
    
    /*
        Collection {
            #items: array:2 [
                0 => "John Doe"
                1 => "Jane Doe"
            ]
        }
    */
    

Si no quiere terminar con la ejecución del script, utilice el método [`dump`](#method-dump).

<a name="method-diff"></a>

#### `diff()` {#collection-method}

El método `diff` compara la colección con otra colección o un `array` de PHP nativo basado en sus valores. Esto método retornará los valores de la colección original que no estén presentes en la colección dada:

    $collection = collect([1, 2, 3, 4, 5]);
    
    $diff = $collection->diff([2, 4, 6, 8]);
    
    $diff->all();
    
    // [1, 3, 5]
    

<a name="method-diffassoc"></a>

#### `diffAssoc()` {#collection-method}

El método `diffAssoc` compara la colección con otra colección o un `array` de PHP nativo basado en sus claves y valores. Este método retornará el par de clave/valor en la colección original que no están presentes en la colección dada:

    $collection = collect([
        'color' => 'orange',
        'type' => 'fruit',
        'remain' => 6
    ]);
    
    $diff = $collection->diffAssoc([
        'color' => 'yellow',
        'type' => 'fruit',
        'remain' => 3,
        'used' => 6
    ]);
    
    $diff->all();
    
    // ['color' => 'orange', 'remain' => 6]
    

<a name="method-diffkeys"></a>

#### `diffKeys()` {#collection-method}

El método `diffKeys` compara la colección con otra colección o un `array` de PHP nativo basado en sus claves. Este método retornará el par de clave/valor en la colección original que no están presentes en la colección dada:

    $collection = collect([
        'one' => 10,
        'two' => 20,
        'three' => 30,
        'four' => 40,
        'five' => 50,
    ]);
    
    $diff = $collection->diffKeys([
        'two' => 2,
        'four' => 4,
        'six' => 6,
        'eight' => 8,
    ]);
    
    $diff->all();
    
    // ['one' => 10, 'three' => 30, 'five' => 50]
    

<a name="method-dump"></a>

#### `dump()` {#collection-method}

El método `dump` vuelca el contenido de los elementos de la colección:

    $collection = collect(['John Doe', 'Jane Doe']);
    
    $collection->dump();
    
    /*
        Collection {
            #items: array:2 [
                0 => "John Doe"
                1 => "Jane Doe"
            ]
        }
    */
    

Si se desea dejar de ejecutar un *script* tras volcar la colección, utilizar el método [`dd`](#method-dd) en su lugar.

<a name="method-each"></a>

#### `each()` {#collection-method}

El método `each` itera sobre los elementos de la colección y pasa cada uno de ellos por un *callback*:

    $collection = $collection->each(function ($item, $key) {
        //
    });
    

Para detener la iteración de elementos, se puede devolver `false` desde el *callback*:

    $collection = $collection->each(function ($item, $key) {
        if (/* some condition */) {
            return false;
        }
    });
    

<a name="method-eachspread"></a>

#### `eachSpread()` {#collection-method}

El método `eachSpread` itera sobre los elementos de la colección, pasando el valor de cada elemento hijo al *callback* dado:

    $collection = collect([['John Doe', 35], ['Jane Doe', 33]]);
    
    $collection->eachSpread(function ($name, $age) {
        //
    });
    

Se puede detener la iteración de elementos retornando `false` desde el *callback*:

    $collection->eachSpread(function ($name, $age) {
        return false;
    });
    

<a name="method-every"></a>

#### `every()` {#collection-method}

El método `every` se puede utilizar para verificar que todos los elementos de una colección pasan una prueba de verdad:

    collect([1, 2, 3, 4])->every(function ($value, $key) {
        return $value > 2;
    });
    
    // false
    

<a name="method-except"></a>

#### `except()` {#collection-method}

El método `except` retorna todos los elementos de la colección excepto aquellos especificados:

    $collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);
    
    $filtered = $collection->except(['price', 'discount']);
    
    $filtered->all();
    
    // ['product_id' => 1]
    

El inverso del método `except` es el método [only](#method-only).

<a name="method-filter"></a>

#### `filter()` {#collection-method}

El método `filter` filtra la colección utilizando un *callback*, únicamente manteniendo aquellos elementos que pasan una prueba de verdad:

    $collection = collect([1, 2, 3, 4]);
    
    $filtered = $collection->filter(function ($value, $key) {
        return $value > 2;
    });
    
    $filtered->all();
    
    // [3, 4]
    

Si no se proporciona un *callback</em, todas las entradas de la colección que sean equivalentes a `false` serán eliminadas:</p> 

    $collection = collect([1, 2, 3, null, false, '', 0, []]);
    
    $collection->filter()->all();
    
    // [1, 2, 3]
    

Para el inverso de `filter`, ver el método [reject](#method-reject).

<a name="method-first"></a>

#### `first()` {#collection-method}

El método `first` retorna el primer elemento de la colección que cumpla la condición:

    collect([1, 2, 3, 4])->first(function ($value, $key) {
        return $value > 2;
    });
    
    // 3
    

También se puede llamar a este método `first` sin argumentos para obtener el primer elemento de la colección. Si la colección está vacía se retornará `null`:

    collect([1, 2, 3, 4])->first();
    
    // 1
    

<a name="method-flatmap"></a>

#### `flatMap()` {#collection-method}

El método `flatMap` itera la colección y pasa cada valor por un *callback* concreto. El *callback* es libre de modificar el elemento y retornarlo, permitiendo formar así una nueva colección de elementos modificados. A continuación el *array* se aplana un nivel:

    $collection = collect([
        ['name' => 'Sally'],
        ['school' => 'Arkansas'],
        ['age' => 28]
    ]);
    
    $flattened = $collection->flatMap(function ($values) {
        return array_map('strtoupper', $values);
    });
    
    $flattened->all();
    
    // ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => '28'];
    

<a name="method-flatten"></a>

#### `flatten()` {#collection-method}

El método `flatten` aplana una colección multidimensional en una única dimensión:

    $collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);
    
    $flattened = $collection->flatten();
    
    $flattened->all();
    
    // ['taylor', 'php', 'javascript'];
    

Opcionalmente se puede pasar un argumento de "profundidad":

    $collection = collect([
        'Apple' => [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
        ],
        'Samsung' => [
            ['name' => 'Galaxy S7', 'brand' => 'Samsung']
        ],
    ]);
    
    $products = $collection->flatten(1);
    
    $products->values()->all();
    
    /*
        [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
            ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
        ]
    */
    

En este ejemplo, llamar a `flatten` sin proporcionar la profundidad aplanará también los *arrays* anidados, resultando en `['iPhone 6S', 'Apple', 'Galaxy S7', 'Samsung']`. Proporcionar la profundidad permite restringir los niveles de *arrays* anidados a aplanar.

<a name="method-flip"></a>

#### `flip()` {#collection-method}

El método `flip` alterna las claves de la colección con sus valores correspondientes:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);
    
    $flipped = $collection->flip();
    
    $flipped->all();
    
    // ['taylor' => 'name', 'laravel' => 'framework']
    

<a name="method-forget"></a>

#### `forget()` {#collection-method}

El método `forget` elimina un elemento de la colección por su clave:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);
    
    $collection->forget('name');
    
    $collection->all();
    
    // ['framework' => 'laravel']
    

> {note} Al contrario que la mayoría de los métodos de las colecciones, `forget` no retorna una colección nueva; modifica la colección en la que se llama.

<a name="method-forpage"></a>

#### `forPage()` {#collection-method}

El método `forPage` retorna una nueva colección conteniendo los elementos que estrían presentes en una página concreta. El método acepta el número de página como primer argumento y el número de elementos a mostrar por página como segundo parámetro:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);
    
    $chunk = $collection->forPage(2, 3);
    
    $chunk->all();
    
    // [4, 5, 6]
    

<a name="method-get"></a>

#### `get()` {#collection-method}

El método `get` retorna el elemento de una clave determinada. Si la clave no existe, retorna `null`:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);
    
    $value = $collection->get('name');
    
    // taylor
    

Opcionalmente se puede pasar el valor por defecto como segundo parámetro:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);
    
    $value = $collection->get('foo', 'default-value');
    
    // default-value
    

Incluso se puede pasar un *callback* como valor por defecto. El resultado del *callback* se retornará si la clave no existe:

    $collection->get('email', function () {
        return 'default-value';
    });
    
    // default-value
    

<a name="method-groupby"></a>

#### `groupBy()` {#collection-method}

El método `groupBy` agrupa los elementos de la colección por una clave dada:

    $collection = collect([
        ['account_id' => 'account-x10', 'product' => 'Chair'],
        ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ['account_id' => 'account-x11', 'product' => 'Desk'],
    ]);
    
    $grouped = $collection->groupBy('account_id');
    
    $grouped->toArray();
    
    /*
        [
            'account-x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'account-x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */
    

Además de pasar una cadena `clave`, se puede pasar un *callback*. El *callback* retornará el valor de la clave para agrupar:

    $grouped = $collection->groupBy(function ($item, $key) {
        return substr($item['account_id'], -3);
    });
    
    $grouped->toArray();
    
    /*
        [
            'x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */
    

<a name="method-has"></a>

#### `has()` {#collection-method}

El método `has` determina si una clave existe en la colección:

    $collection = collect(['account_id' => 1, 'product' => 'Desk']);
    
    $collection->has('product');
    
    // true
    

<a name="method-implode"></a>

#### `implode()` {#collection-method}

El método `implode` unirá los elementos en una colección. Sus argumentos dependen del tipo de elementos en la colección. Si la colección contiene *arrays* u objetos, se debe pasar la clave de los atributos a unir y la cadena de "unión" que se establecerá entre los valores:

    $collection = collect([
        ['account_id' => 1, 'product' => 'Desk'],
        ['account_id' => 2, 'product' => 'Chair'],
    ]);
    
    $collection->implode('product', ', ');
    
    // Desk, Chair
    

Si la colección contiene únicamente cadenas o valores numéricos, simplemente hay que pasar la "unión" como único argumento:

    collect([1, 2, 3, 4, 5])->implode('-');
    
    // '1-2-3-4-5'
    

<a name="method-intersect"></a>

#### `intersect()` {#collection-method}

El método `intersec` elimina cualquier valor de la colección original que no esté presente en un `array` o colección dado. La colección resultante preservará las claves de la colección orignal:

    $collection = collect(['Desk', 'Sofa', 'Chair']);
    
    $intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);
    
    $intersect->all();
    
    // [0 => 'Desk', 2 => 'Chair']
    

<a name="method-intersectbykeys"></a>

#### `intersectByKeys()` {#collection-method}

El método `intersectByKeys` elimina cualquier clave de la colección original que no esté presento en el `array` o colección dados:

    $collection = collect([
        'serial' => 'UX301', 'type' => 'screen', 'year' => 2009
    ]);
    
    $intersect = $collection->intersectByKeys([
        'reference' => 'UX404', 'type' => 'tab', 'year' => 2011
    ]);
    
    $intersect->all();
    
    // ['type' => 'screen', 'year' => 2009]
    

<a name="method-isempty"></a>

#### `isEmpty()` {#collection-method}

El método `isEmpty` retornará `true` si la colección está vacía, de otro modo, retornará `false`:

    collect([])->isEmpty();
    
    // true
    

<a name="method-isnotempty"></a>

#### `isNotEmpty()` {#collection-method}

El método `isNotEmpty` reotrnará `true` si la colección no está vacía; de otro modo, `false`:

    collect([])->isNotEmpty();
    
    // false
    

<a name="method-keyby"></a>

#### `keyBy()` {#collection-method}

El método `keyBy` establece la clave de la colección por una clave concreta. Si hay varios elementos que tienen la misma clave, solo una aparecerá en la nueva colección:

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);
    
    $keyed = $collection->keyBy('product_id');
    
    $keyed->all();
    
    /*
        [
            'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */
    

También se puede pasar un *callback* al método. El *callback* debe retornar el valor de la clave que se desea establecer:

    $keyed = $collection->keyBy(function ($item) {
        return strtoupper($item['product_id']);
    });
    
    $keyed->all();
    
    /*
        [
            'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */
    

<a name="method-keys"></a>

#### `keys()` {#collection-method}

El método `keys` retorna todas las claves de la colección:

    $collection = collect([
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);
    
    $keys = $collection->keys();
    
    $keys->all();
    
    // ['prod-100', 'prod-200']
    

<a name="method-last"></a>

#### `last()` {#collection-method}

El método `last` retorna el último elemento de la colección en cumplir el filtro:

    collect([1, 2, 3, 4])->last(function ($value, $key) {
        return $value < 3;
    });
    
    // 2
    

También se puede llamar al método `last` sin argumentos para obtener el último elemento de la colección. Si la colección está vacía se retornará `null`:

    collect([1, 2, 3, 4])->last();
    
    // 4
    

<a name="method-macro"></a>

#### `macro()` {#collection-method}

El método estático `macro` permite añadir métodos a la clase `Collection` en tiempo de ejecución. Diríjase a la documentación de [extender colecciones](#extending-collections) para más información.

<a name="method-make"></a>

#### `make()` {#collection-method}

El método estático `make` crea una nueva instancia de colección. Ver la sección [crear colecciones](#creating-collections).

<a name="method-map"></a>

#### `map()` {#collection-method}

El método `map` itera la colección y pasa cada elemento al *callback* proporcionado. El *callback* es capaz de modificar el elemento y retornarlo, formando así una nueva colección de elementos modificados:

    $collection = collect([1, 2, 3, 4, 5]);
    
    $multiplied = $collection->map(function ($item, $key) {
        return $item * 2;
    });
    
    $multiplied->all();
    
    // [2, 4, 6, 8, 10]
    

> {note} Como la mayoría de los métodos de la colección, `map` retorna una nueva instancia de la colección; no modifica la colección en la que se ha llamado. Si se desea transformar la colección original, utilizar le método [`transform`](#method-transform).

<a name="method-mapinto"></a>

#### `mapInto()` {#collection-method}

El método `mapInto` itera la colección, creando una nueva instancia de la clave dada pasando el valor al constructor:

    class Currency
    {
        /**
         * Create a new currency instance.
         *
         * @param  string  $code
         * @return void
         */
        function __construct(string $code)
        {
            $this->code = $code;
        }
    }
    
    $collection = collect(['USD', 'EUR', 'GBP']);
    
    $currencies = $collection->mapInto(Currency::class);
    
    $currencies->all();
    
    // [Currency('USD'), Currency('EUR'), Currency('GBP')]
    

<a name="method-mapspread"></a>

#### `mapSpread()` {#collection-method}

El método `mapSpread` itera la colección pasando el valor de cada hijo al *callback* proporcionado. El *callback* es capaz de modificar el elemento y retornarlo, formando así una nueva colección de elementos modificados:

    $collection = collect([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);
    
    $chunks = $collection->chunk(2);
    
    $sequence = $chunks->mapSpread(function ($odd, $even) {
        return $odd + $even;
    });
    
    $sequence->all();
    
    // [1, 5, 9, 13, 17]
    

<a name="method-maptogroups"></a>

#### `mapToGroups()` {#collection-method}

El método `mapToGroups` agrupa los elementos de la colección por un *callback*. El *callback* debe retornar un *array* asociativo conteniendo un par clave/valor, formando así una nueva colección de valores agrupados:

    $collection = collect([
        [
            'name' => 'John Doe',
            'department' => 'Sales',
        ],
        [
            'name' => 'Jane Doe',
            'department' => 'Sales',
        ],
        [
            'name' => 'Johnny Doe',
            'department' => 'Marketing',
        ]
    ]);
    
    $grouped = $collection->mapToGroups(function ($item, $key) {
        return [$item['department'] => $item['name']];
    });
    
    $grouped->toArray();
    
    /*
        [
            'Sales' => ['John Doe', 'Jane Doe'],
            'Marketing' => ['Johhny Doe'],
        ]
    */
    
    $grouped->get('Sales')->all();
    
    // ['John Doe', 'Jane Doe']
    

<a name="method-mapwithkeys"></a>

#### `mapWithKeys()` {#collection-method}

El método `mapWithKeys` itera la colección y pasa cada valor al *callback* dado. El *callback* debe retornar un *array* asociativo conteniendo un único par de clave/valor:

    $collection = collect([
        [
            'name' => 'John',
            'department' => 'Sales',
            'email' => 'john@example.com'
        ],
        [
            'name' => 'Jane',
            'department' => 'Marketing',
            'email' => 'jane@example.com'
        ]
    ]);
    
    $keyed = $collection->mapWithKeys(function ($item) {
        return [$item['email'] => $item['name']];
    });
    
    $keyed->all();
    
    /*
        [
            'john@example.com' => 'John',
            'jane@example.com' => 'Jane',
        ]
    */
    

<a name="method-max"></a>

#### `max()` {#collection-method}

El método `max` retorna el valor máximo de una clave dada:

    $max = collect([['foo' => 10], ['foo' => 20]])->max('foo');
    
    // 20
    
    $max = collect([1, 2, 3, 4, 5])->max();
    
    // 5
    

<a name="method-median"></a>

#### `median()` {#collection-method}

El método `median` retorna el [valor medio](https://en.wikipedia.org/wiki/Median) de una clave concreta:

    $median = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->median('foo');
    
    // 15
    
    $median = collect([1, 1, 2, 4])->median();
    
    // 1.5
    

<a name="method-merge"></a>

#### `merge()` {#collection-method}

El métoo `merge` unifica el *array* o colección dado con la colección original. Si alguna cadena de clave coincide con otra de la colección original, el valor de los elementos se sobrescribirá por el valor de la colección original:

    $collection = collect(['product_id' => 1, 'price' => 100]);
    
    $merged = $collection->merge(['price' => 200, 'discount' => false]);
    
    $merged->all();
    
    // ['product_id' => 1, 'price' => 200, 'discount' => false]
    

Si las claves de los elementos proporcionados son numéricas, los valores se añadirán al final de la colección:

    $collection = collect(['Desk', 'Chair']);
    
    $merged = $collection->merge(['Bookcase', 'Door']);
    
    $merged->all();
    
    // ['Desk', 'Chair', 'Bookcase', 'Door']
    

<a name="method-min"></a>

#### `min()` {#collection-method}

El método `min` retorna el valor mínimo de una clave concreta:

    $min = collect([['foo' => 10], ['foo' => 20]])->min('foo');
    
    // 10
    
    $min = collect([1, 2, 3, 4, 5])->min();
    
    // 1
    

<a name="method-mode"></a>

#### `mode()` {#collection-method}

El método `mode` retorna el [valor de moda](https://en.wikipedia.org/wiki/Mode_(statistics)) de una clave concreta:

    $mode = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->mode('foo');
    
    // [10]
    
    $mode = collect([1, 1, 2, 4])->mode();
    
    // [1]
    

<a name="method-nth"></a>

#### `nth()` {#collection-method}

El método `nth` crea una nueva colección basado en la serie dada:

    $collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);
    
    $collection->nth(4);
    
    // ['a', 'e']
    

Tambíen se puede pasar un <0>offset</0> como segundo argumento:

    $collection->nth(4, 1);
    
    // ['b', 'f']
    

<a name="method-only"></a>

#### `only()` {#collection-method}

El método `only` retorna los elementos de la colección con las claves pasadas:

    $collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);
    
    $filtered = $collection->only(['product_id', 'name']);
    
    $filtered->all();
    
    // ['product_id' => 1, 'name' => 'Desk']
    

El inverso de `only` es el método [except](#method-except).

<a name="method-pad"></a>

#### `pad()` {#collection-method}

El método `pad` rellenará el *array* con el valor dado hasta que el *array* llegue a un tamaño concreto. Este método actua justo como la función de PHP [array_pad](https://secure.php.net/manual/en/function.array-pad.php).

Para rellenar hacia la izquierda, se debe especificar un tamaño negativo. No se hará ningún relleno si el valor absoluto proporcionado es menor o igual a la longitud del *array*:

    $collection = collect(['A', 'B', 'C']);
    
    $filtered = $collection->pad(5, 0);
    
    $filtered->all();
    
    // ['A', 'B', 'C', 0, 0]
    
    $filtered = $collection->pad(-5, 0);
    
    $filtered->all();
    
    // [0, 0, 'A', 'B', 'C']
    

<a name="method-partition"></a>

#### `partition()` {#collection-method}

El método `partition` se puede combinar con la función `lists` de PHP para sepearar elementos que pasan una prueba de verdad de aquellos que no lo hacen:

    $collection = collect([1, 2, 3, 4, 5, 6]);
    
    list($underThree, $aboveThree) = $collection->partition(function ($i) {
        return $i < 3;
    });
    

<a name="method-pipe"></a>

#### `pipe()` {#collection-method}

El método `pipe` pasa la colección por un *callback* y retorna el resultado:

    $collection = collect([1, 2, 3]);
    
    $piped = $collection->pipe(function ($collection) {
        return $collection->sum();
    });
    
    // 6
    

<a name="method-pluck"></a>

#### `pluck()` {#collection-method}

El método `pluck` obtiene todos los valores para una clave dada:

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);
    
    $plucked = $collection->pluck('name');
    
    $plucked->all();
    
    // ['Desk', 'Chair']
    

Además se puede especificar qué clave se debe utilizar para la colección resultante:

    $plucked = $collection->pluck('name', 'product_id');
    
    $plucked->all();
    
    // ['prod-100' => 'Desk', 'prod-200' => 'Chair']
    

<a name="method-pop"></a>

#### `pop()` {#collection-method}

El método `pop` elimina y retorna el último elemento de la colección:

    $collection = collect([1, 2, 3, 4, 5]);
    
    $collection->pop();
    
    // 5
    
    $collection->all();
    
    // [1, 2, 3, 4]
    

<a name="method-prepend"></a>

#### `prepend()` {#collection-method}

El método `prepend` añade un elemento al principio de la colección:

    $collection = collect([1, 2, 3, 4, 5]);
    
    $collection->prepend(0);
    
    $collection->all();
    
    // [0, 1, 2, 3, 4, 5]
    

También se puede psar un segundo parámetro para establecer la clave del elemento a añadir:

    $collection = collect(['one' => 1, 'two' => 2]);
    
    $collection->prepend(0, 'zero');
    
    $collection->all();
    
    // ['zero' => 0, 'one' => 1, 'two' => 2]
    

<a name="method-pull"></a>

#### `pull()` {#collection-method}

El método `pull` elimina y retorna un elemento de la colección por su clave:

    $collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);
    
    $collection->pull('name');
    
    // 'Desk'
    
    $collection->all();
    
    // ['product_id' => 'prod-100']
    

<a name="method-push"></a>

#### `push()` {#collection-method}

El método `push` añade un elemento al final de la colección:

    $collection = collect([1, 2, 3, 4]);
    
    $collection->push(5);
    
    $collection->all();
    
    // [1, 2, 3, 4, 5]
    

<a name="method-put"></a>

#### `put()` {#collection-method}

El método `put` establece una clave y valor en la colección:

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);
    
    $collection->put('price', 100);
    
    $collection->all();
    
    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100]
    

<a name="method-random"></a>

#### `random()` {#collection-method}

El método `random` retorna un elemento aleatorio de la colección:

    $collection = collect([1, 2, 3, 4, 5]);
    
    $collection->random();
    
    // 4 - (retrieved randomly)
    

De forma opcional, se puede pasar un entero a `random` para especificar cuantos elementos se desea obtener de forma aleatoria. Siempre se retornará una colección de elementos cuando explícitamente se pase el número de elementos que se desea obtener:

    $random = $collection->random(3);
    
    $random->all();
    
    // [2, 4, 5] - (retrieved randomly)
    

<a name="method-reduce"></a>

#### `reduce()` {#collection-method}

El método `reduce` reduce la colección a un único valor, pasando el resultado de cada iteración a la iteración subsecuente:

    $collection = collect([1, 2, 3]);
    
    $total = $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    });
    
    // 6
    

El valor de `$carry` en la primera iteración es `null`; sin embargo, se puede especificar su valor inicial pasándole un segundo argumento a `reduce`:

    $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    }, 4);
    
    // 10
    

<a name="method-reject"></a>

#### `reject()` {#collection-method}

El método `reject` filtra la colección utilizando un *callback*. El *callback* debe retornar `true` si el elemento debe eliminarse de la colección resultante:

    $collection = collect([1, 2, 3, 4]);
    
    $filtered = $collection->reject(function ($value, $key) {
        return $value > 2;
    });
    
    $filtered->all();
    
    // [1, 2]
    

El inverso del método `reject` es el método [`filter`](#method-filter).

<a name="method-reverse"></a>

#### `reverse()` {#collection-method}

El método `reverse` invierte el orden de los elementos de la colección, preservando las claves originales:

    $collection = collect(['a', 'b', 'c', 'd', 'e']);
    
    $reversed = $collection->reverse();
    
    $reversed->all();
    
    /*
        [
            4 => 'e',
            3 => 'd',
            2 => 'c',
            1 => 'b',
            0 => 'a',
        ]
    */
    

<a name="method-search"></a>

#### `search()` {#collection-method}

El método de `search` busca en la colección el valor dado y devuelve su clave si la encuentra. Si el elemento no se encuentra, se devuelve `false`.

    $collection = collect([2, 4, 6, 8]);
    
    $collection->search(4);
    
    // 1
    

La búsqueda se realiza utilizando una comparación "débil" (*loose*), por lo que una cadena con un entero se considerará igual que un entero del mismo valor. Para utilizar una comparación "estricta", indicar `true` como segundo parámetro:

    $collection->search('4', true);
    
    // false
    

De forma alternativa, se puede pasar un *callback* propio a la búsqueda para encontrar el primer elemento que cumpla la condición:

    $collection->search(function ($item, $key) {
        return $item > 5;
    });
    
    // 2
    

<a name="method-shift"></a>

#### `shift()` {#collection-method}

El método `shift` elimina y retorna el primer elemento de la colección:

    $collection = collect([1, 2, 3, 4, 5]);
    
    $collection->shift();
    
    // 1
    
    $collection->all();
    
    // [2, 3, 4, 5]
    

<a name="method-shuffle"></a>

#### `shuffle()` {#collection-method}

El método `shuffle` mezcla al azar los elementos de la colección:

    $collection = collect([1, 2, 3, 4, 5]);
    
    $shuffled = $collection->shuffle();
    
    $shuffled->all();
    
    // [3, 2, 5, 1, 4] - (generated randomly)
    

<a name="method-slice"></a>

#### `slice()` {#collection-method}

El método `slice` retorna una parte de la colección comenzando por un índice dado:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);
    
    $slice = $collection->slice(4);
    
    $slice->all();
    
    // [5, 6, 7, 8, 9, 10]
    

Se puede pasar el tamaño como segundo argumento al método:

    $slice = $collection->slice(4, 2);
    
    $slice->all();
    
    // [5, 6]
    

La porción resultante preservará las claves por defecto. Si no se desea preservar las claves originales, se puede utilizar el método [`values`](#method-values) para *reindexar* la colección.

<a name="method-sort"></a>

#### `sort()` {#collection-method}

El método `sort` oordena la colección. La colección ordenada mantiene las claves originales, por lo que en este ejemplo se utilizará el método [`values`](#method-values) para restablecer las claves a índices consecutivos:

    $collection = collect([5, 3, 1, 2, 4]);
    
    $sorted = $collection->sort();
    
    $sorted->values()->all();
    
    // [1, 2, 3, 4, 5]
    

Si las necesidades de órden son más avanzadas, se puede pasar un *callback* a `sort` con un algoritmo propio. Refiérase a la documentación de PHP sobre [`uasort`](https://secure.php.net/manual/en/function.uasort.php#refsect1-function.uasort-parameters), la cual usa el método `sort` bajo el capó.

> {tip} Si se necesita ordenar una colección de *arrays* u objetos anidados, ver los métodos [`sortBy`](#method-sortby) y [`sortByDesc`](#method-sortbydesc).

<a name="method-sortby"></a>

#### `sortBy()` {#collection-method}

El método `sortBy` ordena la colección por una clave dada. La colección ordenada mantiene las claves originales, por lo que en este ejemplo se utilizará el método [`values`](#method-values) para restablecer las claves a índices consecutivos:

    $collection = collect([
        ['name' => 'Desk', 'price' => 200],
        ['name' => 'Chair', 'price' => 100],
        ['name' => 'Bookcase', 'price' => 150],
    ]);
    
    $sorted = $collection->sortBy('price');
    
    $sorted->values()->all();
    
    /*
        [
            ['name' => 'Chair', 'price' => 100],
            ['name' => 'Bookcase', 'price' => 150],
            ['name' => 'Desk', 'price' => 200],
        ]
    */
    

También se puede pasar un *callback* propio para determinar como ordenar los valores de la colección:

    $collection = collect([
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);
    
    $sorted = $collection->sortBy(function ($product, $key) {
        return count($product['colors']);
    });
    
    $sorted->values()->all();
    
    /*
        [
            ['name' => 'Chair', 'colors' => ['Black']],
            ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
            ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
        ]
    */
    

<a name="method-sortbydesc"></a>

#### `sortByDesc()` {#collection-method}

Este método tiene la misma firma que el método de [`sortBy`](#method-sortby), pero ordenará a la colección en orden inverso.

<a name="method-splice"></a>

#### `splice()` {#collection-method}

El método `splice` elimina y retorna una porción de elementos comenzando desde un índice específico:

    $collection = collect([1, 2, 3, 4, 5]);
    
    $chunk = $collection->splice(2);
    
    $chunk->all();
    
    // [3, 4, 5]
    
    $collection->all();
    
    // [1, 2]
    

Se puede pasar un segundo argumento para limitar el tamaño de la porción resultante:

    $collection = collect([1, 2, 3, 4, 5]);
    
    $chunk = $collection->splice(2, 1);
    
    $chunk->all();
    
    // [3]
    
    $collection->all();
    
    // [1, 2, 4, 5]
    

Además, un tercero que contiene los nuevos elementos para reemplazar a los eliminados de la colección:

    $collection = collect([1, 2, 3, 4, 5]);
    
    $chunk = $collection->splice(2, 1, [10, 11]);
    
    $chunk->all();
    
    // [3]
    
    $collection->all();
    
    // [1, 2, 10, 11, 4, 5]
    

<a name="method-split"></a>

#### `split()` {#collection-method}

El método `split` divide la colección en tantos grupos como se indique:

    $collection = collect([1, 2, 3, 4, 5]);
    
    $groups = $collection->split(3);
    
    $groups->toArray();
    
    // [[1, 2], [3, 4], [5]]
    

<a name="method-sum"></a>

#### `sum()` {#collection-method}

El método `sum` suma todos los elementos de la colección:

    collect([1, 2, 3, 4, 5])->sum();
    
    // 15
    

Si la colección contiene *arrays* anidados u objetos, se debe pasar una clave para determinar qué valores se han de sumar:

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);
    
    $collection->sum('pages');
    
    // 1272
    

Además, se puede pasar un *callback* propio para determinar qué elementos sumar:

    $collection = collect([
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);
    
    $collection->sum(function ($product) {
        return count($product['colors']);
    });
    
    // 6
    

<a name="method-take"></a>

#### `take()` {#collection-method}

El método `take` retorna una colección con un número específico de elementos:

    $collection = collect([0, 1, 2, 3, 4, 5]);
    
    $chunk = $collection->take(3);
    
    $chunk->all();
    
    // [0, 1, 2]
    

También se puede pasar un valor negativo para tomar esa cantidad de elementos desde el final de la colección:

    $collection = collect([0, 1, 2, 3, 4, 5]);
    
    $chunk = $collection->take(-2);
    
    $chunk->all();
    
    // [4, 5]
    

<a name="method-tap"></a>

#### `tap()` {#collection-method}

El método `tap` pasa la colección por un *callback* proporcionado, permitiendo "tocar/alterar" la colección en un punto concreto y hacer algo con sus elementos sin afectar a la colección en sí misma:

    collect([2, 4, 3, 1, 5])
        ->sort()
        ->tap(function ($collection) {
            Log::debug('Values after sorting', $collection->values()->toArray());
        })
        ->shift();
    
    // 1
    

<a name="method-times"></a>

#### `times()` {#collection-method}

El método estático `times` crea una nueva colección invocando el *callback* la cantidad de veces dada:

    $collection = Collection::times(10, function ($number) {
        return $number * 9;
    });
    
    $collection->all();
    
    // [9, 18, 27, 36, 45, 54, 63, 72, 81, 90]
    

Este método puede ser útil al combinarlo con factorías para crear modelos [Eloquent](/docs/{{version}}/eloquent):

    $categories = Collection::times(3, function ($number) {
        return factory(Category::class)->create(['name' => 'Category #'.$number]);
    });
    
    $categories->all();
    
    /*
        [
            ['id' => 1, 'name' => 'Category #1'],
            ['id' => 2, 'name' => 'Category #2'],
            ['id' => 3, 'name' => 'Category #3'],
        ]
    */
    

<a name="method-toarray"></a>

#### `toArray()` {#collection-method}

El método `toArray` convierte la colección en un `array` PHP plano. Si los elementos de la colección son modelos [Eloquent](/docs/{{version}}/eloquent), los modelos también se convertirán a *arrays*:

    $collection = collect(['name' => 'Desk', 'price' => 200]);
    
    $collection->toArray();
    
    /*
        [
            ['name' => 'Desk', 'price' => 200],
        ]
    */
    

> {note} `toArray` convierte además los objetos anidados a un *array*. Si se desea obtener el *array* subyacente, utilizar el método [`all`](#method-all) en su lugar.

<a name="method-tojson"></a>

#### `toJson()` {#collection-method}

El método `toJson` convierte la colección en una cadena JSON *serializada*:

    $collection = collect(['name' => 'Desk', 'price' => 200]);
    
    $collection->toJson();
    
    // '{"name":"Desk", "price":200}'
    

<a name="method-transform"></a>

#### `transform()` {#collection-method}

El método `transform` itera sobre todos los elementos de la colección y llama al *callback* con cada uno de ellos. Los elementos de la colección se reemplazarán con los valores retornados por el *callback*:

    $collection = collect([1, 2, 3, 4, 5]);
    
    $collection->transform(function ($item, $key) {
        return $item * 2;
    });
    
    $collection->all();
    
    // [2, 4, 6, 8, 10]
    

> {note} Al contrario que la mayoría de los métodos de las colecciones, `transform` modifica la colección en si misma. Para crear una nueva colección, utilizar el método [`map`](#method-map) en su lugar.

<a name="method-union"></a>

#### `union()` {#collection-method}

El método `union` añade el *array* proporcionado a la colección. Si el *array* contiene claves que ya existen en la colección, se preferirán los valores de la colección original:

    $collection = collect([1 => ['a'], 2 => ['b']]);
    
    $union = $collection->union([3 => ['c'], 1 => ['b']]);
    
    $union->all();
    
    // [1 => ['a'], 2 => ['b'], 3 => ['c']]
    

<a name="method-unique"></a>

#### `unique()` {#collection-method}

El método `unique` retorna todos los elementos únicos de la colección. La colección resultante mantendrá las claves originales, por lo que en este ejemplo se utilizará el método [`values`](#method-values) para restablecer las claves a valores consecutivos:

    $collection = collect([1, 1, 2, 2, 3, 4, 2]);
    
    $unique = $collection->unique();
    
    $unique->values()->all();
    
    // [1, 2, 3, 4]
    

Cuando se trata con *arrays* anidados u objetos, se puede especificar la clave que determina la unicidad:

    $collection = collect([
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'iPhone 5', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
    ]);
    
    $unique = $collection->unique('brand');
    
    $unique->values()->all();
    
    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ]
    */
    

También se puede pasar un *callback* propio para determinar la unicidad:

    $unique = $collection->unique(function ($item) {
        return $item['brand'].$item['type'];
    });
    
    $unique->values()->all();
    
    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
            ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
        ]
    */
    

El método `unique` utiliza una comparación "débil" (*loose*) al comprobar los valores de los elementos, por lo que uan cadena con un valor entero se considerará igual que un entero del mismo valor. Utilice el método [`uniqueStrict`](#method-uniquestrict) para filtrar utilizando comparaciones "estrictas".

<a name="method-uniquestrict"></a>

#### `uniqueStrict()` {#collection-method}

Este método tiene la misma firma que le método [`unique`](#method-unique); sin embargo, todos los valores se comparan utilizando comparaciones "estrictas".

<a name="method-unless"></a>

#### `unless()` {#collection-method}

El método `unless` ejecutará el *callback* dado a no ser que el primer argumento del método se resuelva a `true`:

    $collection = collect([1, 2, 3]);
    
    $collection->unless(true, function ($collection) {
        return $collection->push(4);
    });
    
    $collection->unless(false, function ($collection) {
        return $collection->push(5);
    });
    
    $collection->all();
    
    // [1, 2, 3, 5]
    

El inverso de `unless` sería el método [`when`](#method-when).

<a name="method-unwrap"></a>

#### `unwrap()` {#collection-method}

El método estático `unwrap` retorna los elementos subyacentes de la colección para el valor dado cuando aplique:

    Collection::unwrap(collect('John Doe'));
    
    // ['John Doe']
    
    Collection::unwrap(['John Doe']);
    
    // ['John Doe']
    
    Collection::unwrap('John Doe');
    
    // 'John Doe'
    

<a name="method-values"></a>

#### `values()` {#collection-method}

El método `values` retorna una colección con las claves restablecidas a enteros consecutivos:

    $collection = collect([
        10 => ['product' => 'Desk', 'price' => 200],
        11 => ['product' => 'Desk', 'price' => 200]
    ]);
    
    $values = $collection->values();
    
    $values->all();
    
    /*
        [
            0 => ['product' => 'Desk', 'price' => 200],
            1 => ['product' => 'Desk', 'price' => 200],
        ]
    */
    

<a name="method-when"></a>

#### `when()` {#collection-method}

El método `when` ejecutará el *callback* proporcionado cuando el primer argumento se resuelva a `true`:

    $collection = collect([1, 2, 3]);
    
    $collection->when(true, function ($collection) {
        return $collection->push(4);
    });
    
    $collection->when(false, function ($collection) {
        return $collection->push(5);
    });
    
    $collection->all();
    
    // [1, 2, 3, 4]
    

El inverso de `when` sería el método [`unless`](#method-unless).

<a name="method-where"></a>

#### `where()` {#collection-method}

El método `where` filtra la colección por un par de clave / valor:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);
    
    $filtered = $collection->where('price', 100);
    
    $filtered->all();
    
    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */
    

El método `where` utiliza una comparación "débil" (*loose*) al comprobar los valores de los elementos, por lo que uan cadena con un valor entero se considerará igual que un entero del mismo valor. Utilice el método [`whereStrict`](#method-wherestrict) para filtrar utilizando comparaciones "estrictas".

<a name="method-wherestrict"></a>

#### `whereStrict()` {#collection-method}

Este método tiene la misma firma que le método [`where`](#method-where); sin embargo, todos los valores se comparan utilizando comparaciones "estrictas".

<a name="method-wherein"></a>

#### `whereIn()` {#collection-method}

El método `whereIn` filtra la colección por un par clave/valor dado contenido dentro del *array* de elementos:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);
    
    $filtered = $collection->whereIn('price', [150, 200]);
    
    $filtered->all();
    
    /*
        [
            ['product' => 'Bookcase', 'price' => 150],
            ['product' => 'Desk', 'price' => 200],
        ]
    */
    

El método `whereIn` utiliza una comparación "débil" (*loose*) al comprobar los valores de los elementos, por lo que uan cadena con un valor entero se considerará igual que un entero del mismo valor. Utilice el método [`whereInStrict`](#method-whereinstrict) para filtrar utilizando comparaciones "estrictas".

<a name="method-whereinstrict"></a>

#### `whereInStrict()` {#collection-method}

Este método tiene la misma firma que le método [`whereIn`](#method-wherein); sin embargo, todos los valores se comparan utilizando comparaciones "estrictas".

<a name="method-wherenotin"></a>

#### `whereNotIn()` {#collection-method}

El método `whereNotIn` filtra la colección por un par clave/valor dado que no exista dentro del *array* de elementos:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);
    
    $filtered = $collection->whereNotIn('price', [150, 200]);
    
    $filtered->all();
    
    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */
    

El método `whereNotIn` utiliza una comparación "débil" (*loose*) al comprobar los valores de los elementos, por lo que uan cadena con un valor entero se considerará igual que un entero del mismo valor. Utilice el método [`whereNotInStrict`](#method-wherenotinstrict) para filtrar utilizando comparaciones "estrictas".

<a name="method-wherenotinstrict"></a>

#### `whereNotInStrict()` {#collection-method}

Este método tiene la misma firma que le método [`whereNotIn`](#method-wherenotin); sin embargo, todos los valores se comparan utilizando comparaciones "estrictas".

<a name="method-wrap"></a>

#### `wrap()` {#collection-method}

El método `wrap` envuelve el valor proporcionado en una colección si es posible:

    $collection = Collection::wrap('John Doe');
    
    $collection->all();
    
    // ['John Doe']
    
    $collection = Collection::wrap(['John Doe']);
    
    $collection->all();
    
    // ['John Doe']
    
    $collection = Collection::wrap(collect('John Doe'));
    
    $collection->all();
    
    // ['John Doe']
    

<a name="method-zip"></a>

#### `zip()` {#collection-method}

El método `zip` mezcla juntos los valores del *array* dado con los valores de la colección original basado en su índice:

    $collection = collect(['Chair', 'Desk']);
    
    $zipped = $collection->zip([100, 200]);
    
    $zipped->all();
    
    // [['Chair', 100], ['Desk', 200]]
    

<a name="higher-order-messages"></a>

## Nivel superior de mensajes

Las colecciones soportan además "niveles superiores de mensajes" o "Higher Order Messages (Hom)", los cuales son atajos para ejecutar acciones comunes en colecciones. Los métodos de la colección que soportan esta característica son: `average`, `avg`, `contains`, `each`, `every`, `filter`, `first`, `flatMap`, `map`, `partition`, `reject`, `sortBy`, `sortByDesc`, y `sum`.

Cada mensaje se puede acceder como una propiedad dinámica en una instancia de una colección. Por ejemplo, el mensage `each` se puede utilizar para llamar a un método por cada elemento en la colección:

    $users = User::where('votes', '>', 500)->get();
    
    $users->each->markAsVip();
    

Del mismo modo, se puede utilizar el mensaje de nivel superior `sum` para calcular el número total de votos de una colección de usuarios:

    $users = User::where('group', 'Development')->get();
    
    return $users->sum->votes;