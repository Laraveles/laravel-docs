# *Helpers*

- [Introducción](#introduction)
- [Métodos disponibles](#available-methods)

<a name="introduction"></a>

## Introducción

Laravel incluye una gran variedad de funciones "*helper*". Muchas de estas funciones son utilizadas por el propio framework; sin embargo, es libre de usarlas en sus propias aplicaciones si lo considera conveniente.

<a name="available-methods"></a>

## Métodos disponibles

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

### Arrays & objetos

<div class="collection-method-list">
  <p>
    <a href="#method-array-add">array_add</a> <a href="#method-array-collapse">array_collapse</a> <a href="#method-array-divide">array_divide</a> <a href="#method-array-dot">array_dot</a> <a href="#method-array-except">array_except</a> <a href="#method-array-first">array_first</a> <a href="#method-array-flatten">array_flatten</a> <a href="#method-array-forget">array_forget</a> <a href="#method-array-get">array_get</a> <a href="#method-array-has">array_has</a> <a href="#method-array-last">array_last</a> <a href="#method-array-only">array_only</a> <a href="#method-array-pluck">array_pluck</a> <a href="#method-array-prepend">array_prepend</a> <a href="#method-array-pull">array_pull</a> <a href="#method-array-random">array_random</a> <a href="#method-array-set">array_set</a> <a href="#method-array-sort">array_sort</a> <a href="#method-array-sort-recursive">array_sort_recursive</a> <a href="#method-array-where">array_where</a> <a href="#method-array-wrap">array_wrap</a> <a href="#method-data-fill">data_fill</a> <a href="#method-data-get">data_get</a> <a href="#method-data-set">data_set</a> <a href="#method-head">head</a> <a href="#method-last">last</a>
  </p>
</div>

### Rutas

<div class="collection-method-list">
  <p>
    <a href="#method-app-path">app_path</a> <a href="#method-base-path">base_path</a> <a href="#method-config-path">config_path</a> <a href="#method-database-path">database_path</a> <a href="#method-mix">mix</a> <a href="#method-public-path">public_path</a> <a href="#method-resource-path">resource_path</a> <a href="#method-storage-path">storage_path</a>
  </p>
</div>

### Cadenas

<div class="collection-method-list">
  <p>
    <a href="#method-__">\__</a> <a href="#method-camel-case">camel_case</a> <a href="#method-class-basename">class_basename</a> <a href="#method-e">e</a> <a href="#method-ends-with">ends_with</a> <a href="#method-kebab-case">kebab_case</a> <a href="#method-preg-replace-array">preg_replace_array</a> <a href="#method-snake-case">snake_case</a> <a href="#method-starts-with">starts_with</a> <a href="#method-str-after">str_after</a> <a href="#method-str-before">str_before</a> <a href="#method-str-contains">str_contains</a> <a href="#method-str-finish">str_finish</a> <a href="#method-str-is">str_is</a> <a href="#method-str-limit">str_limit</a> <a href="#method-str-plural">str_plural</a> <a href="#method-str-random">str_random</a> <a href="#method-str-replace-array">str_replace_array</a> <a href="#method-str-replace-first">str_replace_first</a> <a href="#method-str-replace-last">str_replace_last</a> <a href="#method-str-singular">str_singular</a> <a href="#method-str-slug">str_slug</a> <a href="#method-str-start">str_start</a> <a href="#method-studly-case">studly_case</a> <a href="#method-title-case">title_case</a> <a href="#method-trans">trans</a> <a href="#method-trans-choice">trans_choice</a>
  </p>
</div>

### URLs

<div class="collection-method-list">
  <p>
    <a href="#method-action">action</a> <a href="#method-asset">asset</a> <a href="#method-secure-asset">secure_asset</a> <a href="#method-route">route</a> <a href="#method-secure-url">secure_url</a> <a href="#method-url">url</a>
  </p>
</div>

### Varios

<div class="collection-method-list">
  <p>
    <a href="#method-abort">abort</a> <a href="#method-abort-if">abort_if</a> <a href="#method-abort-unless">abort_unless</a> <a href="#method-app">app</a> <a href="#method-auth">auth</a> <a href="#method-back">back</a> <a href="#method-bcrypt">bcrypt</a> <a href="#method-blank">blank</a> <a href="#method-broadcast">broadcast</a> <a href="#method-cache">cache</a> <a href="#method-class-uses-recursive">class_uses_recursive</a> <a href="#method-collect">collect</a> <a href="#method-config">config</a> <a href="#method-cookie">cookie</a> <a href="#method-csrf-field">csrf_field</a> <a href="#method-csrf-token">csrf_token</a> <a href="#method-dd">dd</a> <a href="#method-decrypt">decrypt</a> <a href="#method-dispatch">dispatch</a> <a href="#method-dispatch-now">dispatch_now</a> <a href="#method-dump">dump</a> <a href="#method-encrypt">encrypt</a> <a href="#method-env">env</a> <a href="#method-event">event</a> <a href="#method-factory">factory</a> <a href="#method-filled">filled</a> <a href="#method-info">info</a> <a href="#method-logger">logger</a> <a href="#method-method-field">method_field</a> <a href="#method-now">now</a> <a href="#method-old">old</a> <a href="#method-optional">optional</a> <a href="#method-policy">policy</a> <a href="#method-redirect">redirect</a> <a href="#method-report">report</a> <a href="#method-request">request</a> <a href="#method-rescue">rescue</a> <a href="#method-resolve">resolve</a> <a href="#method-response">response</a> <a href="#method-retry">retry</a> <a href="#method-session">session</a> <a href="#method-tap">tap</a> <a href="#method-today">today</a> <a href="#method-throw-if">throw_if</a> <a href="#method-throw-unless">throw_unless</a> <a href="#method-trait-uses-recursive">trait_uses_recursive</a> <a href="#method-transform">transform</a> <a href="#method-validator">validator</a> <a href="#method-value">value</a> <a href="#method-view">view</a> <a href="#method-with">with</a>
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

<a name="arrays"></a>

## Arrays & objetos

<a name="method-array-add"></a>

#### `array_add()` {#collection-method.first-collection-method}

La función `array_add` añade un par clave/valor dado a un *array* si la clave dada no existe ya en el mismo:

    $array = array_add(['name' => 'Desk'], 'price', 100);
    
    // ['name' => 'Desk', 'price' => 100]
    

<a name="method-array-collapse"></a>

#### `array_collapse()` {#collection-method}

La función `array_collapse` colapsa un *array* de *arrays* en un solo *array*:

    $array = array_collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);
    
    // [1, 2, 3, 4, 5, 6, 7, 8, 9]
    

<a name="method-array-divide"></a>

#### `array_divide()` {#collection-method}

La función `array_divide` devuelve dos *arrays*, uno que contiene las claves y el otro que contiene los valores del *array* en cuestión:

    list($keys, $values) = array_divide(['name' => 'Desk']);
    
    // $keys: ['name']
    
    // $values: ['Desk']
    

<a name="method-array-dot"></a>

#### `array_dot()` {#collection-method}

La función `array_dot` convierte un *array* multidimensional en un *array* de un único nivel utilizando la notación de "puntos" para indicar la profundidad:

    $array = ['products' => ['desk' => ['price' => 100]]];
    
    $flattened = array_dot($array);
    
    // ['products.desk.price' => 100]
    

<a name="method-array-except"></a>

#### `array_except()` {#collection-method}

La función `array_except` elimina los pares clave/valor dados de un *array*:

    $array = ['name' => 'Desk', 'price' => 100];
    
    $filtered = array_except($array, ['price']);
    
    // ['name' => 'Desk']
    

<a name="method-array-first"></a>

#### `array_first()` {#collection-method}

El método `array_first` devuelve el primer elemento de un *array* que cumpla una condición dada:

    $array = [100, 200, 300];
    
    $first = array_first($array, function ($value, $key) {
        return $value >= 150;
    });
    
    // 200
    

Se puede pasar un valor por defecto como tercer parámetro al método. Este valor se devolverá si no hay un valor que cumpla la condición:

    $first = array_first($array, $callback, $default);
    

<a name="method-array-flatten"></a>

#### `array_flatten()` {#collection-method}

La función `array_flatten` aplana un *array* multidimensional en un *array* de un solo nivel:

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];
    
    $flattened = array_flatten($array);
    
    // ['Joe', 'PHP', 'Ruby']
    

<a name="method-array-forget"></a>

#### `array_forget()` {#collection-method}

El método `array_forget` elimina un par clave/valor dado de un *array* anidado utilizando la notación de "puntos":

    $array = ['products' => ['desk' => ['price' => 100]]];
    
    array_forget($array, 'products.desk');
    
    // ['products' => []]
    

<a name="method-array-get"></a>

#### `array_get()` {#collection-method}

El método `array_get` recupera un valor de un *array* anidado usando la notación de "puntos":

    $array = ['products' => ['desk' => ['price' => 100]]];
    
    $price = array_get($array, 'products.desk.price');
    
    // 100
    

La función `array_get` también acepta un valor predeterminado, que será devuelto si no se encuentra la clave especificada:

    $discount = array_get($array, 'products.desk.discount', 0);
    
    // 0
    

<a name="method-array-has"></a>

#### `array_has()` {#collection-method}

La función `array_has` comprueba si un elemento o elementos determinados existen en un *array* utilizando la notación de "puntos":

    $array = ['product' => ['name' => 'Desk', 'price' => 100]];
    
    $contains = array_has($array, 'product.name');
    
    // true
    
    $contains = array_has($array, ['product.price', 'product.discount']);
    
    // false
    

<a name="method-array-last"></a>

#### `array_last()` {#collection-method}

La función `array_last` devuelve el último elemento de un *array* que pasa una prueba de verdad concreta:

    $array = [100, 200, 300, 110];
    
    $last = array_last($array, function ($value, $key) {
        return $value >= 150;
    });
    
    // 300
    

Se puede pasar un valor predeterminado como tercer argumento al método. Este valor se devolverá si ningún valor pasa la prueba de la verdad:

    $last = array_last($array, $callback, $default);
    

<a name="method-array-only"></a>

#### `array_only()` {#collection-method}

La función `array_only` devuelve sólo los pares clave/valor especificados del *array* dado:

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];
    
    $slice = array_only($array, ['name', 'price']);
    
    // ['name' => 'Desk', 'price' => 100]
    

<a name="method-array-pluck"></a>

#### `array_pluck()` {#collection-method}

La función `array_pluck` recupera todos los valores de una clave determinada de un *array*:

    $array = [
        ['developer' => ['id' => 1, 'name' => 'Taylor']],
        ['developer' => ['id' => 2, 'name' => 'Abigail']],
    ];
    
    $names = array_pluck($array, 'developer.name');
    
    // ['Taylor', 'Abigail']
    

También se puede especificar que clave obtendrá el resultado:

    $names = array_pluck($array, 'developer.name', 'developer.id');
    
    // [1 => 'Taylor', 2 => 'Abigail']
    

<a name="method-array-prepend"></a>

#### `array_prepend()` {#collection-method}

La función `array_prepend` añadirá un elemento al principio del *array*:

    $array = ['one', 'two', 'three', 'four'];
    
    $array = array_prepend($array, 'zero');
    
    // ['zero', 'one', 'two', 'three', 'four']
    

Si es necesario, puede especificar la clave que debe utilizarse para el valor:

    $array = ['price' => 100];
    
    $array = array_prepend($array, 'Desk', 'name');
    
    // ['name' => 'Desk', 'price' => 100]
    

<a name="method-array-pull"></a>

#### `array_pull()` {#collection-method}

La función `array_pull` devuelve y elimina un par clave/valor del *array*:

    $array = ['name' => 'Desk', 'price' => 100];
    
    $name = array_pull($array, 'name');
    
    // $name: Desk
    
    // $array: ['price' => 100]
    

Se puede pasar un valor predeterminado como tercer argumento al método. Este valor se devolverá si la clave no existe:

    $value = array_pull($array, $key, $default);
    

<a name="method-array-random"></a>

#### `array_random()` {#collection-method}

La función `array_random` devuelve un valor aleatorio del *array*:

    $array = [1, 2, 3, 4, 5];
    
    $random = array_random($array);
    
    // 4 - (retrieved randomly)
    

También puede especificar el número de elementos a devolver como segundo argumento opcional. Tenga en cuenta que al proporcionar este argumento se devolverá un *array*, incluso si sólo se desea un elemento:

    $items = array_random($array, 2);
    
    // [2, 5] - (retrieved randomly)
    

<a name="method-array-set"></a>

#### `array_set()` {#collection-method}

El método `array_set` establece un valor dentro de un *array* anidado en profundidad mediante la notación de "puntos":

    $array = ['products' => ['desk' => ['price' => 100]]];
    
    array_set($array, 'products.desk.price', 200);
    
    // ['products' => ['desk' => ['price' => 200]]]
    

<a name="method-array-sort"></a>

#### `array_sort()` {#collection-method}

La función `array_sort` ordena un *array* por sus valores:

    $array = ['Desk', 'Table', 'Chair'];
    
    $sorted = array_sort($array);
    
    // ['Chair', 'Desk', 'Table']
    

También puede ordenar el *array* por los resultados del *Closure* dado:

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Table'],
        ['name' => 'Chair'],
    ];
    
    $sorted = array_values(array_sort($array, function ($value) {
        return $value['name'];
    }));
    
    /*
        [
            ['name' => 'Chair'],
            ['name' => 'Desk'],
            ['name' => 'Table'],
        ]
    */
    

<a name="method-array-sort-recursive"></a>

#### `array_sort_recursive()` {#collection-method}

La función `array_sort_recursive` un *array* de forma recursiva usando la función `sort`:

    $array = [
        ['Roman', 'Taylor', 'Li'],
        ['PHP', 'Ruby', 'JavaScript'],
    ];
    
    $sorted = array_sort_recursive($array);
    
    /*
        [
            ['Li', 'Roman', 'Taylor'],
            ['JavaScript', 'PHP', 'Ruby'],
        ]
    */
    

<a name="method-array-where"></a>

#### `array_where()` {#collection-method}

La función `array_where` filtra un *array* usando el *Closure* dado:

    $array = [100, '200', 300, '400', 500];
    
    $filtered = array_where($array, function ($value, $key) {
        return is_string($value);
    });
    
    // [1 => 200, 3 => 400]
    

<a name="method-array-wrap"></a>

#### `array_wrap()` {#collection-method}

La función `array_wrap` envuelve el valor dado en un *array*. Si el valor dado ya es un *array* no se modificará:

    $string = 'Laravel';
    
    $array = array_wrap($string);
    
    // ['Laravel']
    

<a name="method-data-fill"></a>

#### `data_fill()` {#collection-method}

La función `data_fill` establece un valor faltante dentro del *array* anidado u objeto usando la notación de "punto":

    $data = ['products' => ['desk' => ['price' => 100]]];
    
    data_fill($data, 'products.desk.price', 200);
    
    // ['products' => ['desk' => ['price' => 100]]]
    
    data_fill($data, 'products.desk.discount', 10);
    
    // ['products' => ['desk' => ['price' => 100, 'discount' => 10]]]
    

Esta función también acepta asteriscos como comodines y rellenará el objetivo en consecuencia:

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2'],
        ],
    ];
    
    data_fill($data, 'products.*.price', 200);
    
    /*
        [
            'products' => [
                ['name' => 'Desk 1', 'price' => 100],
                ['name' => 'Desk 2', 'price' => 200],
            ],
        ]
    */
    

<a name="method-data-get"></a>

#### `data_get()` {#collection-method}

La función `data_get` recupera un valor de un *array* anidado u objeto utilizando la notación de "puntos":

    $data = ['products' => ['desk' => ['price' => 100]]];
    
    $price = data_get($data, 'products.desk.price');
    
    // 100
    

La función `data_get` también acepta un valor predeterminado, que se devolverá si no se encuentra la clave especificada:

    $discount = data_get($data, 'products.desk.discount', 0);
    
    // 0
    

<a name="method-data-set"></a>

#### `data_set()` {#collection-method}

La función `data_set` establece un valor dentro de un *array* anidado u objeto usando la notación de "punto":

    $data = ['products' => ['desk' => ['price' => 100]]];
    
    data_set($data, 'products.desk.price', 200);
    
    // ['products' => ['desk' => ['price' => 200]]]
    

Esta función también acepta comodines y fijará los valores en el objetivo en consecuencia:

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2', 'price' => 150],
        ],
    ];
    
    data_set($data, 'products.*.price', 200);
    
    /*
        [
            'products' => [
                ['name' => 'Desk 1', 'price' => 200],
                ['name' => 'Desk 2', 'price' => 200],
            ],
        ]
    */
    

Por defecto, todos los valores existentes se sobrescriben. Si sólo desea establecer un valor si no existe, puede pasar `false` como tercer argumento:

    $data = ['products' => ['desk' => ['price' => 100]]];
    
    data_set($data, 'products.desk.price', 200, false);
    
    // ['products' => ['desk' => ['price' => 100]]]
    

<a name="method-head"></a>

#### `head()` {#collection-method}

La función `head` devuelve el primer elemento del *array*:

    $array = [100, 200, 300];
    
    $first = head($array);
    
    // 100
    

<a name="method-last"></a>

#### `last()` {#collection-method}

La función `last` devuelve el último elemento del *array* especificado:

    $array = [100, 200, 300];
    
    $last = last($array);
    
    // 300
    

<a name="paths"></a>

## Rutas

<a name="method-app-path"></a>

#### `app_path()` {#collection-method}

La función `app_path` devuelve la ruta cualificada al directorio `app`. También puede utilizar la función `app_path` para generar una ruta totalmente cualificada a un archivo relativo al directorio *app*:

    $path = app_path();
    
    $path = app_path('Http/Controllers/Controller.php');
    

<a name="method-base-path"></a>

#### `base_path()` {#collection-method}

La función `base_path` devuelve la ruta cualificada a la raíz del proyecto. También puede utilizar la función `base_path` para generar una ruta totalmente cualificada a un archivo determinado en relación con el directorio raíz del proyecto:

    $path = base_path();
    
    $path = base_path('vendor/bin');
    

<a name="method-config-path"></a>

#### `config_path()` {#collection-method}

La función `config_path` devuelve la ruta totalmente cualificada al directorio `config`. También puede utilizar la función `config_path` para generar una ruta totalmente cualificada a un archivo determinado dentro del directorio *config*:

    $path = config_path();
    
    $path = config_path('app.php');
    

<a name="method-database-path"></a>

#### `database_path()` {#collection-method}

La función `database_path` devuelve la ruta totalmente cualificada al directorio `database`. También puede utilizar la función `database_path` para generar una ruta totalmente cualificada a un archivo determinado dentro del directorio *database*:

    $path = database_path();
    
    $path = database_path('factories/UserFactory.php');
    

<a name="method-mix"></a>

#### `mix()` {#collection-method}

La función `mix` devuelve la ruta a un [archivo de *Mix* versionado](/docs/{{version}}/mix):

    $path = mix('css/app.css');
    

<a name="method-public-path"></a>

#### `public_path()` {#collection-method}

La función `public_path` devuelve la ruta totalmente cualificada al directorio `public`. También puede utilizar la función `public_path` para generar una ruta totalmente cualificada a un archivo determinado dentro del directorio *public*:

    $path = public_path();
    
    $path = public_path('css/app.css');
    

<a name="method-resource-path"></a>

#### `resource_path()` {#collection-method}

La función `resource_path` devuelve la ruta totalmente cualificada al directorio `resources`. También puede utilizar la función `resource_path` para generar una ruta totalmente cualificada a un archivo determinado dentro del directorio *resources*:

    $path = resource_path();
    
    $path = resource_path('assets/sass/app.scss');
    

<a name="method-storage-path"></a>

#### `storage_path()` {#collection-method}

La función `storage_path` devuelve la ruta totalmente cualificada al directorio `storage`. También puede utilizar la función `storage_path` para generar una ruta totalmente cualificada a un archivo determinado dentro del directorio *storage*:

    $path = storage_path();
    
    $path = storage_path('app/file.txt');
    

<a name="strings"></a>

## Cadenas

<a name="method-__"></a>

#### `__()` {#collection-method}

La función `__` traduce la cadena de traducción o la clave de traducción utilizando sus [ficheros de localización](/docs/{{version}}/localization):

    echo __('Welcome to our application');
    
    echo __('messages.welcome');
    

Si la cadena o clave de traducción especificada no existe, la función `__` simplemente devolverá el valor proporcionado. Por lo tanto, utilizando el ejemplo anterior, la función `__` devolvería `messages.welcome` si esa clave de conversión no existe.

<a name="method-camel-case"></a>

#### `camel_case()` {#collection-method}

La función `camel_case` convierte la cadena dada en `camelCase`:

    $converted = camel_case('foo_bar');
    
    // fooBar
    

<a name="method-class-basename"></a>

#### `class_basename()` {#collection-method}

`class_basename` devuelve el nombre de la clase especificada sin el *namespace* de la propia clase:

    $class = class_basename('Foo\Bar\Baz');
    
    // Baz
    

<a name="method-e"></a>

#### `e()` {#collection-method}

La función `e` ejecuta la función `htmlspecialchars` de PHP con la opción `double_encode` ajustada a `false`:

    echo e('<html>foo</html>');
    
    // &lt;html&gt;foo&lt;/html&gt;
    

<a name="method-ends-with"></a>

#### `ends_with()` {#collection-method}

La función `ends_with` determina si la cadena dada termina con un valor especificado:

    $result = ends_with('This is my name', 'name');
    
    // true
    

<a name="method-kebab-case"></a>

#### `kebab_case()` {#collection-method}

La función `kebab_case` convierte la cadena dada a `kebab-case`:

    $converted = kebab_case('fooBar');
    
    // foo-bar
    

<a name="method-preg-replace-array"></a>

#### `preg_replace_array()` {#collection-method}

La función `preg_replace_array` reemplaza un patrón dado en la cadena de forma secuencial utilizando un *array*:

    $string = 'The event will take place between :start and :end';
    
    $replaced = preg_replace_array('/:[a-z_]+/', ['8:30', '9:00'], $string);
    
    // The event will take place between 8:30 and 9:00
    

<a name="method-snake-case"></a>

#### `snake_case()` {#collection-method}

La función `snake_case` convierte la cadena dada en formato `snake_case`:

    $converted = snake_case('fooBar');
    
    // foo_bar
    

<a name="method-starts-with"></a>

#### `starts_with()` {#collection-method}

La función `starts_with` determina si la cadena dada comienza con el valor especificado:

    $result = starts_with('This is my name', 'This');
    
    // true
    

<a name="method-str-after"></a>

#### `str_after()` {#collection-method}

La función `str_after` devuelve todo después del valor dado en una cadena:

    $slice = str_after('This is my name', 'This is');
    
    // ' my name'
    

<a name="method-str-before"></a>

#### `str_before()` {#collection-method}

La función `str_before` devuelve todo antes del valor dado en una cadena:

    $slice = str_before('This is my name', 'my name');
    
    // 'This is '
    

<a name="method-str-contains"></a>

#### `str_contains()` {#collection-method}

La función `str_contains` determina si la cadena dada contiene el valor especificado:

    $contains = str_contains('This is my name', 'my');
    
    // true
    

También puede pasar un *array* de valores para determinar si la cadena contiene alguno de ellos:

    $contains = str_contains('This is my name', ['my', 'foo']);
    
    // true
    

<a name="method-str-finish"></a>

#### `str_finish()` {#collection-method}

La función `str_finish` añade una sola instancia del valor dado a una cadena si no termina ya con el valor:

    $adjusted = str_finish('this/string', '/');
    
    // this/string/
    
    $adjusted = str_finish('this/string/', '/');
    
    // this/string/
    

<a name="method-str-is"></a>

#### `str_is()` {#collection-method}

La función `str_is` determina si una cadena dada coincide con un patrón determinado. Se pueden utilizar asteriscos para indicar comodines:

    $matches = str_is('foo*', 'foobar');
    
    // true
    
    $matches = str_is('baz*', 'foobar');
    
    // false
    

<a name="method-str-limit"></a>

#### `str_limit()` {#collection-method}

La función `str_limit` interrumpe la cadena en la longitud especificada:

    $truncated = str_limit('The quick brown fox jumps over the lazy dog', 20);
    
    // The quick brown fox...
    

También puede pasar un tercer argumento para cambiar la cadena que se agregará al final:

    $truncated = str_limit('The quick brown fox jumps over the lazy dog', 20, ' (...)');
    
    // The quick brown fox (...)
    

<a name="method-str-plural"></a>

#### `str_plural()` {#collection-method}

La función `str_plural` convierte una cadena a su forma plural. Esta función actualmente sólo soporta el idioma inglés:

    $plural = str_plural('car');
    
    // cars
    
    $plural = str_plural('child');
    
    // children
    

Se puede proporcionar a la función un número entero como segundo argumento para recuperar la forma singular o plural de la cadena:

    $plural = str_plural('child', 2);
    
    // children
    
    $plural = str_plural('child', 1);
    
    // child
    

<a name="method-str-random"></a>

#### `str_random()` {#collection-method}

La función `str_random` genera una cadena aleatoria de la longitud especificada. Utiliza la función de PHP `random_bytes`:

    $random = str_random(40);
    

<a name="method-str-replace-array"></a>

#### `str_replace_array()` {#collection-method}

La función `str_replace_array` reemplaza un valor dado en la cadena secuencialmente usando un *array*:

    $string = 'The event will take place between ? and ?';
    
    $replaced = str_replace_array('?', ['8:30', '9:00'], $string);
    
    // The event will take place between 8:30 and 9:00
    

<a name="method-str-replace-first"></a>

#### `str_replace_first()` {#collection-method}

La función `str_replace_first` reemplaza la primera aparición de un valor dado en una cadena:

    $replaced = str_replace_first('the', 'a', 'the quick brown fox jumps over the lazy dog');
    
    // a quick brown fox jumps over the lazy dog
    

<a name="method-str-replace-last"></a>

#### `str_replace_last()` {#collection-method}

La función `str_replace_last` reemplaza la última aparición de un valor dado en una cadena:

    $replaced = str_replace_last('the', 'a', 'the quick brown fox jumps over the lazy dog');
    
    // the quick brown fox jumps over a lazy dog
    

<a name="method-str-singular"></a>

#### `str_singular()` {#collection-method}

La función `str_singular` convierte una cadena a su forma singular. Esta función actualmente sólo soporta el idioma inglés:

    $singular = str_singular('cars');
    
    // car
    
    $singular = str_singular('children');
    
    // child
    

<a name="method-str-slug"></a>

#### `str_slug()` {#collection-method}

La función `str_slug` genera un "slug" amigable para la URL de la cadena especificada:

    $slug = str_slug('Laravel 5 Framework', '-');
    
    // laravel-5-framework
    

<a name="method-str-start"></a>

#### `str_start()` {#collection-method}

La función `str_start` añade una sola instancia del valor dado a una cadena si no comienza ya con el valor:

    $adjusted = str_start('this/string', '/');
    
    // /this/string
    
    $adjusted = str_start('/this/string/', '/');
    
    // /this/string
    

<a name="method-studly-case"></a>

#### `studly_case()` {#collection-method}

La función `studly_case` convierte la cadena dada en formato `StudlyCase`:

    $converted = studly_case('foo_bar');
    
    // FooBar
    

<a name="method-title-case"></a>

#### `title_case()` {#collection-method}

La función `title_case` convierte la cadena dada a `Title Case`:

    $converted = title_case('a nice title uses the correct case');
    
    // A Nice Title Uses The Correct Case
    

<a name="method-trans"></a>

#### `trans()` {#collection-method}

La función `trans` traduce la clave de conversión dada utilizando los [archivos de localización](/docs/{{version}}/localization):

    echo trans('messages.welcome');
    

Si la clave de conversión especificada no existe, la función `trans` simplemente devolverá la clave dada. Por lo tanto, utilizando el ejemplo anterior, la función `trans` devolvería `messages.welcome` si la clave de traducción no existe.

<a name="method-trans-choice"></a>

#### `trans_choice()` {#collection-method}

La función `trans_choice` traduce la clave de conversión dada con inflexión:

    echo trans_choice('messages.notifications', $unreadCount);
    

Si la clave de conversión especificada no existe, la función `trans_choice` simplemente devolverá la clave dada. Por lo tanto, utilizando el ejemplo anterior, la función `trans_choice` devolvería `messages.notifications` si no existe la clave de conversión.

<a name="urls"></a>

## URLs

<a name="method-action"></a>

#### `action()` {#collection-method}

La función `action` genera una dirección URL para una acción determinada de un controlador. No es necesario especificar el *namespace* completo del controlador. En vez de eso, hay que pasar el nombre de la clase relativo al *namespace* `App\Http\Controllers`:

    $url = action('HomeController@index');
    

Si el método acepta parámetros de ruta, se pueden pasar como segundo argumento al método:

    $url = action('UserController@profile', ['id' => 1]);
    

<a name="method-asset"></a>

#### `asset()` {#collection-method}

La función `asset` genera una URL para un *asset* utilizando el esquema actual de la petición (HTTP o HTTPS):

    $url = asset('img/photo.jpg');
    

<a name="method-secure-asset"></a>

#### `secure_asset()` {#collection-method}

La función `secure_asset` genera una URL para un *asset* usando HTTPS:

    $url = secure_asset('img/photo.jpg');
    

<a name="method-route"></a>

#### `route()` {#collection-method}

La función `route` genera una URL para un nombre de ruta dado:

    $url = route('routeName');
    

Si la ruta acepta parámetros, se pueden pasar como segundo argumento al método:

    $url = route('routeName', ['id' => 1]);
    

De forma predeterminada, la función `route` genera una URL absoluta. Si desea generar una URL relativa, puede pasar `false` como tercer argumento:

    $url = route('routeName', ['id' => 1], false);
    

<a name="method-secure-url"></a>

#### `secure_url()` {#collection-method}

La función `secure_url` genera una URL HTTPS totalmente cualificada a la ruta dada:

    $url = secure_url('user/profile');
    
    $url = secure_url('user/profile', [1]);
    

<a name="method-url"></a>

#### `url()` {#collection-method}

La función `url` genera una URL completa a la ruta dada:

    $url = url('user/profile');
    
    $url = url('user/profile', [1]);
    

Si no se proporciona ninguna ruta, se devuelve una instancia de `Illuminate\Routing\UrlGenerator`:

    $current = url()->current();
    
    $full = url()->full();
    
    $previous = url()->previous();
    

<a name="miscellaneous"></a>

## Varios

<a name="method-abort"></a>

#### `abort()` {#collection-method}

La función `abort` lanza [una excepción HTTP](/docs/{{version}}/errors#http-exceptions) que será mostrada por el [gestor de excepciones](/docs/{{version}}/errors#the-exception-handler):

    abort(403);
    

También puede proporcionar el texto de respuesta de la excepción y encabezados de respuesta personalizados:

    abort(403, 'Unauthorized.', $headers);
    

<a name="method-abort-if"></a>

#### `abort_if()` {#collection-method}

La función `abort_if` lanza una excepción HTTP si una expresión *booleana* especificada se evalúa a `true`:

    abort_if(! Auth::user()->isAdmin(), 403);
    

Al igual que el método `abort`, también puede proporcionar el texto de respuesta de la excepción como tercer argumento y un *array* de cabeceras de respuesta personalizadas como cuarto argumento.

<a name="method-abort-unless"></a>

#### `abort_unless()` {#collection-method}

La función `abort_unless` lanza una excepción HTTP si una expresión *booleana* dada evalúa `false`:

    abort_unless(Auth::user()->isAdmin(), 403);
    

Al igual que el método `abort`, también puede proporcionar el texto de respuesta de la excepción como tercer argumento y una matriz de encabezados de respuesta personalizados como cuarto argumento.

<a name="method-app"></a>

#### `app()` {#collection-method}

La función `app` devuelve una instancia de [service container](/docs/{{version}}/container):

    $container = app();
    

Puede pasar un nombre de clase o interfaz para resolverlo desde el contenedor:

    $api = app('HelpSpot\API');
    

<a name="method-auth"></a>

#### `auth()` {#collection-method}

La función `auth` devuelve una instancia de [authenticator](/docs/{{version}}/authentication). Puede utilizarlo en lugar de la *facade* `Auth` por conveniencia:

    $user = auth()->user();
    

Si es necesario, puede especificar a qué instancia de *guard* desea acceder:

    $user = auth('admin')->user();
    

<a name="method-back"></a>

#### `back()` {#collection-method}

La función `back` genera una [respuesta HTTP redireccionada](/docs/{{version}}/responses#redirects) a la ubicación anterior del usuario:

    return back($status = 302, $headers = [], $fallback = false);
    
    return back();
    

<a name="method-bcrypt"></a>

#### `bcrypt()` {#collection-method}

La función `bcrypt` [hashea](/docs/{{version}}/hashing) el valor dado utilizando Bcrypt. Se puede utilizar como alternativa a la *facade* `Hash`:

    $password = bcrypt('my-secret-password');
    

<a name="method-broadcast"></a>

#### `broadcast()` {#collection-method}

La función `broadcast` [difunde ("broadcasts")](/docs/{{version}}/broadcasting) [el evento](/docs/{{version}}/events) dado a sus *listeners*:

    broadcast(new UserRegistered($user));
    

<a name="method-blank"></a>

#### `blank()` {#collection-method}

La función `blank` devuelve si el valor dado está "en blanco":

    blank('');
    blank('   ');
    blank(null);
    blank(collect());
    
    // true
    
    blank(0);
    blank(true);
    blank(false);
    
    // false
    

Para el inverso de `blank`, vea el método [`filled`](#method-filled).

<a name="method-cache"></a>

#### `cache()` {#collection-method}

La función `cache` puede usarse para obtener valores de la [caché](/docs/{{version}}/cache). Si la clave dada no existe en la caché, se devolverá un valor predeterminado opcional:

    $value = cache('key');
    
    $value = cache('key', 'default');
    

Puede añadir elementos a la caché pasando una matriz de pares clave/valor a la función. También debe pasar el número de minutos o la duración que el valor almacenado en caché debe considerarse válido:

    cache(['key' => 'value'], 5);
    
    cache(['key' => 'value'], Carbon::now()->addSeconds(10));
    

<a name="method-class-uses-recursive"></a>

#### `class_uses_recursive()` {#collection-method}

La función `class_uses_recursive` devuelve todos los *traits* usados por una clase, incluyendo los *traits* usados por cualquier subclase:

    $traits = class_uses_recursive(App\User::class);
    

<a name="method-collect"></a>

#### `collect()` {#collection-method}

La función `collect` crea una instancia de [collection](/docs/{{version}}/collections) a partir del valor dado:

    $collection = collect(['taylor', 'abigail']);
    

<a name="method-config"></a>

#### `config()` {#collection-method}

La función `config` obtiene el valor de una variable de [configuración](/docs/{{version}}/configuration). Los valores de configuración pueden accederse mediante una sintaxis de "puntos", que incluye el nombre del archivo y la opción a acceder. Se puede especificar un valor por defecto para ser devuelto si no existe la opción de configuración:

    $value = config('app.timezone');
    
    $value = config('app.timezone', $default);
    

Puede establecer variables de configuración en tiempo de ejecución pasando un *array* de pares clave/valor:

    config(['app.debug' => true]);
    

<a name="method-cookie"></a>

#### `cookie()` {#collection-method}

La función `cookie` crea una nueva instancia de [cookie](/docs/{{version}}/requests#cookies):

    $cookie = cookie('name', 'value', $minutes);
    

<a name="method-csrf-field"></a>

#### `csrf_field()` {#collection-method}

La función `csrf_field` genera un campo HTML input `hidden` que contiene el valor del *token CSRF*. Por ejemplo, utilizando [sintaxis de Blade](/docs/{{version}}/blade):

    {{ csrf_field() }}
    

<a name="method-csrf-token"></a>

#### `csrf_token()` {#collection-method}

La función `csrf_token` recupera el valor del *token CSRF* actual:

    $token = csrf_token();
    

<a name="method-dd"></a>

#### `dd()` {#collection-method}

La función `dd` muestra información sobre las variables dadas y termina la ejecución del script:

    dd($value);
    
    dd($value1, $value2, $value3, ...);
    

Si no desea detener la ejecución de su script, utilice la función [`dump`](#method-dump).

<a name="method-decrypt"></a>

#### `decrypt()` {#collection-method}

La función `decrypt` desencripta el valor dado utilizando el [encrypter](/docs/{{version}}/encryption) de Laravel:

    $decrypted = decrypt($encrypted_value);
    

<a name="method-dispatch"></a>

#### `dispatch()` {#collection-method}

La función `dispatch` añade el [trabajo](/docs/{{version}}/queues#creating-jobs) dado a la [cola de trabajo](/docs/{{version}}/queues) de Laravel:

    dispatch(new App\Jobs\SendEmails);
    

<a name="method-dispatch-now"></a>

#### `dispatch_now()` {#collection-method}

La función `dispatch_now` ejecuta el [trabajo](/docs/{{version}}/queues#creating-jobs) inmediatamente y devuelve el valor de su método `handle`:

    $result = dispatch_now(new App\Jobs\SendEmails);
    

<a name="method-dump"></a>

#### `dump()` {#collection-method}

La función `dump` muestra información sobre las variables dadas:

    dump($value);
    
    dump($value1, $value2, $value3, ...);
    

Si desea detener la ejecución del script después de haber mostrado información sobre las variables, utilice la función [`dd`](#method-dd) en su lugar.

<a name="method-encrypt"></a>

#### `encrypt()` {#collection-method}

La función `encrypt` encripta el valor dado usando el [encrypter](/docs/{{version}}/encryption) de Laravel:

    $encrypted = encrypt($unencrypted_value);
    

<a name="method-env"></a>

#### `env()` {#collection-method}

La función `env` recupera el valor de una [variable del entorno](/docs/{{version}}/configuration#environment-configuration) o devuelve un valor por defecto:

    $env = env('APP_ENV');
    
    // Returns 'production' if APP_ENV is not set...
    $env = env('APP_ENV', 'production');
    

<a name="method-event"></a>

#### `event()` {#collection-method}

La función `event` lanza un [evento](/docs/{{version}}/events) a sus *listeners*:

    event(new UserRegistered($user));
    

<a name="method-factory"></a>

#### `factory()` {#collection-method}

La función `factory` crea un *model factory builder* para una clase, nombre y cantidad. Puede ser utilizado durante [testing](/docs/{{version}}/database-testing#writing-factories) (pruebas) o [seeding](/docs/{{version}}/seeding#using-model-factories) (poblado):

    $user = factory(App\User::class)->make();
    

<a name="method-filled"></a>

#### `filled()` {#collection-method}

La función `filled` devuelve si el valor dado no está "en blanco":

    filled(0);
    filled(true);
    filled(false);
    
    // true
    
    filled('');
    filled('   ');
    filled(null);
    filled(collect());
    
    // false
    

Para el inverso de `filled`, ver el método [`blank`](#method-blank).

<a name="method-info"></a>

#### `info()` {#collection-method}

La función `info` escribirá información en el [log](/docs/{{version}}/errors#logging):

    info('Some helpful information!');
    

También se puede transferir a la función un array de datos contextuales:

    info('User login attempt failed.', ['id' => $user->id]);
    

<a name="method-logger"></a>

#### `logger()` {#collection-method}

La función `logger` puede utilizarse para escribir un mensaje de nivel `debug` en el [log](/docs/{{version}}/errors#logging):

    logger('Debug message');
    

También se puede transferir a la función un *array* de datos contextuales:

    logger('User has logged in.', ['id' => $user->id]);
    

Se devolverá una instancia de [logger](/docs/{{version}}/errors#logging) si no se pasa ningún valor a la función:

    logger()->error('You are not allowed here.');
    

<a name="method-method-field"></a>

#### `method_field()` {#collection-method}

La función `method_field` genera un campo HTML *input* `hidden` que almacena el valor del verbo HTTP del formulario. Por ejemplo, utilizando la [sintaxis Blade](/docs/{{version}}/blade):

    <form method="POST">
        {{ method_field('DELETE') }}
    </form>
    

<a name="method-now"></a>

#### `now()` {#collection-method}

La función `now` crea una nueva instancia de `Illuminate\Support\Carbon` para la hora actual:

    $now = now();
    

<a name="method-old"></a>

#### `old()` {#collection-method}

La función `old` [recupera](/docs/{{version}}/requests#retrieving-input) un valor de [entrada antiguo](/docs/{{version}}/requests#old-input) de la sesión:

    $value = old('value');
    
    $value = old('value', 'default');
    

<a name="method-optional"></a>

#### `optional()` {#collection-method}

La función `optional` acepta cualquier argumento y le permite acceder a las propiedades o métodos de llamada de ese objeto. Si el objeto dado es `null`, las propiedades y métodos simplemente devolverán `null` en lugar de causar un error:

    return optional($user->address)->street;
    
    {!! old('name', optional($user)->name) !!}
    

<a name="method-policy"></a>

#### `policy()` {#collection-method}

El método `policy` recupera una instancia de [policy](/docs/{{version}}/authorization#creating-policies) para una clase determinada:

    $policy = policy(App\User::class);
    

<a name="method-redirect"></a>

#### `redirect()` {#collection-method}

La función `redirect` devuelve una [respuesta HTTP de redirección](/docs/{{version}}/responses#redirects), o devuelve la instancia del redireccionador si se llama sin argumentos:

    return redirect($to = null, $status = 302, $headers = [], $secure = null);
    
    return redirect('/home');
    
    return redirect()->route('route.name');
    

<a name="method-report"></a>

#### `report()` {#collection-method}

La función `report` reportará una excepción usando el método `report` de su [manejador de excepciones](/docs/{{version}}/errors#the-exception-handler):

    report($e);
    

<a name="method-request"></a>

#### `request()` {#collection-method}

La función `request` devuelve la instancia actual de [request](/docs/{{version}}/requests) u obtiene un elemento de entrada:

    $request = request();
    
    $value = request('key', $default = null);
    

<a name="method-rescue"></a>

#### `rescue()` {#collection-method}

La función `rescue` ejecuta el *Closure* dado y captura cualquier excepción que ocurra durante su ejecución. Todas las excepciones que sean capturadas se enviarán al método `report` del [gestor de excepciones](/docs/{{version}}/errors#the-exception-handler); sin embargo, la solicitud continuará procesándose:

    return rescue(function () {
        return $this->method();
    });
    

También puede pasar un segundo argumento a la función `rescue`. Este argumento será el valor "por defecto" que debe devolverse si se produce una excepción durante la ejecución del *Closure*:

    return rescue(function () {
        return $this->method();
    }, false);
    
    return rescue(function () {
        return $this->method();
    }, function () {
        return $this->failure();
    });
    

<a name="method-resolve"></a>

#### `resolve()` {#collection-method}

La función `resolve` resuelve un determinado nombre de clase o interfaz a su instancia utilizando el [contenedor de servicio](/docs/{{version}}/container):

    $api = resolve('HelpSpot\API');
    

<a name="method-response"></a>

#### `response()` {#collection-method}

La función `response` crea una instancia de [response](/docs/{{version}}/responses) u obtiene una instancia de una factoría *response*:

    return response('Hello World', 200, $headers);
    
    return response()->json(['foo' => 'bar'], 200, $headers);
    

<a name="method-retry"></a>

#### `retry()` {#collection-method}

La función `retry` intenta ejecutar la llamada de retorno hasta que se alcanza el umbral de intento máximo. Si la devolución de llamada no lanza una excepción, se devolverá su valor de retorno. Si la devolución de llamada lanza una excepción, se volverá a repetir automáticamente. Si se excede el número máximo de intentos, la excepción será lanzada:

    return retry(5, function () {
        // Attempt 5 times while resting 100ms in between attempts...
    }, 100);
    

<a name="method-session"></a>

#### `session()` {#collection-method}

La función `session` se puede utilizar para obtener o establecer valores de [sesión](/docs/{{version}}/session):

    $value = session('key');
    

Se pueden establecer valores pasando un *array* con el par clave/valor a la función:

    session(['chairs' => 7, 'instruments' => 3]);
    

La sesión almacenada se retornará si no se pasa ningún valor a la función:

    $value = session()->get('key');
    
    session()->put('key', $value);
    

<a name="method-tap"></a>

#### `tap()` {#collection-method}

La función `tap` acepta dos argumentos: un valor `$value` y un *Closure*. El `$value` pasará al *Closure* y luego será devuelto por la función `tap`. El valor de retorno del *Closure* es irrelevante:

    $user = tap(User::first(), function ($user) {
        $user->name = 'taylor';
    
        $user->save();
    });
    

Si no se pasa ningún *Closure* a la función `tap`, puede llamar a cualquier método en el `$value` dado. El valor de retorno del método que llama siempre será `$value`, independientemente de lo que el método devuelva realmente en su definición. Por ejemplo, el método Eloquent `update` normalmente devuelve un entero. Sin embargo, se puede forzar al método a devolver el modelo mismo encadenando la llamada `update` a través de la función `tap`:

    $user = tap($user)->update([
        'name' => $name,
        'email' => $email,
    ]);
    

<a name="method-today"></a>

#### `today()` {#collection-method}

La función `today` crea una nueva instancia de `Illuminate\Support\Carbon` para la fecha actual:

    $today = today();
    

<a name="method-throw-if"></a>

#### `throw_if()` {#collection-method}

La función `throw_if` lanza la excepción dada si una expresión booleana determinada evalúa `true`:

    throw_if(! Auth::user()->isAdmin(), AuthorizationException::class);
    
    throw_if(
        ! Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page'
    );
    

<a name="method-throw-unless"></a>

#### `throw_unless()` {#collection-method}

La función `throw_unless` lanza la excepción dada si una expresión booleana dada evalúa `false`:

    throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);
    
    throw_unless(
        Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page'
    );
    

<a name="method-trait-uses-recursive"></a>

#### `trait_uses_recursive()` {#collection-method}

La función `trait_uses_recursive` devuelve todos los *traits* usados por un *trait*:

    $traits = trait_uses_recursive(\Illuminate\Notifications\Notifiable::class);
    

<a name="method-transform"></a>

#### `transform()` {#collection-method}

La función `transform` ejecuta un `Closure` en un valor dado si no está en [blanco](#method-blank) y devuelve el resultado del `Closure`:

    $callback = function ($value) {
        return $value * 2;
    };
    
    $result = transform(5, $callback);
    
    // 10
    

También se puede pasar al método un valor predeterminado o `Closure` como tercer parámetro. Este valor se devolverá si el valor dado está en blanco:

    $result = transform(null, $callback, 'The value is blank');
    
    // The value is blank
    

<a name="method-validator"></a>

#### `validator()` {#collection-method}

La función `validator` crea una nueva instancia de [validator](/docs/{{version}}/validation) con los argumentos dados. Puede utilizarlo en lugar de la *facade* `Validador` por conveniencia:

    $validator = validator($data, $rules, $messages);
    

<a name="method-value"></a>

#### `value()` {#collection-method}

La función `value` devuelve el valor indicado. Sin embargo, si se proporciona un `Closure` a la función, se devolverá el resultado del `Closure`:

    $result = value(true);
    
    // true
    
    $result = value(function () {
        return false;
    });
    
    // false
    

<a name="method-view"></a>

#### `view()` {#collection-method}

La función `view` retorna una instancia de [view](/docs/{{version}}/views):

    return view('auth.login');
    

<a name="method-with"></a>

#### `with()` {#collection-method}

La función `with` devuelve el valor indicado. Si se pasa un `Closure` como segundo argumento a la función, se ejecutará `Closure` y se devolverá el resultado:

    $callback = function ($value) {
        return (is_numeric($value)) ? $value * 2 : 0;
    };
    
    $result = with(5, $callback);
    
    // 10
    
    $result = with(null, $callback);
    
    // 0
    
    $result = with(5, null);
    
    // 5