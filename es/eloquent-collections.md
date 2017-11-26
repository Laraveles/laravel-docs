# Eloquent: Colecciones

- [Introducción](#introduction)
- [Métodos disponibles](#available-methods)
- [Colecciones personalizadas](#custom-collections)

<a name="introduction"></a>

## Introducción

Todos los conjuntos multi-resultado que devuelve Eloquent son instancias de `Illuminate\Database\Eloquent\Collection`, incluyendo los resultados que se obtienen a través del método `get` o que se acceden a través de una relación. La colección Eloquent hereda de [Collection](/docs/{{version}}/collections), por lo que hereda docenas de métodos que se utilizarán para trabajar de forma fluida con el *array* de modelos Eloquent subyacente.

Por supuesto, todas las colecciones sirven además como "iteradores", permitiendo iterar sobre ellas como si fueran *arrays* de PHP normales:

    $users = App\User::where('active', 1)->get();
    
    foreach ($users as $user) {
        echo $user->name;
    }
    

Sin embargo, las colecciones son mucho más potentes que los *arrays* y exponen una gran variedad de operaciones de mapeo / reducción que pueden ser encadenadas utilizando una interfaz muy intuitiva. Por ejemplo, para eliminar todos los modelos inactivos y obtener el nombre de cada usuario restante:

    $users = App\User::where('active', 1)->get();
    
    $names = $users->reject(function ($user) {
        return $user->active === false;
    })
    ->map(function ($user) {
        return $user->name;
    });
    

> {note} Mientras que la mayoría de los métodos de las colecciones Eloquent devuelven una nueva instancia de la colección, los métodos `pluck`, `keys`, `zip`, `collapse`, `flatten` y `flip` retornan una instancia de la [colección base](/docs/{{version}}/collections). Así mismo, si una operación `map` retorna una colección que no contiene ningún modelo Eloquent, será automáticamente convertida a una colección normal.

<a name="available-methods"></a>

## Métodos disponibles

### La colección base

Todas las colecciones Eloquent heredan del objeto [colección de Laravel](/docs/{{version}}/collections); por tanto, heredan todos los potentes métodos que provee esta clase base:

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
    <a href="/docs/{{version}}/collections#method-all">all</a> <a href="/docs/{{version}}/collections#method-average">average</a> <a href="/docs/{{version}}/collections#method-avg">avg</a> <a href="/docs/{{version}}/collections#method-chunk">chunk</a> <a href="/docs/{{version}}/collections#method-collapse">collapse</a> <a href="/docs/{{version}}/collections#method-combine">combine</a> <a href="/docs/{{version}}/collections#method-concat">concat</a> <a href="/docs/{{version}}/collections#method-contains">contains</a> <a href="/docs/{{version}}/collections#method-containsstrict">containsStrict</a> <a href="/docs/{{version}}/collections#method-count">count</a> <a href="/docs/{{version}}/collections#method-crossjoin">crossJoin</a> <a href="/docs/{{version}}/collections#method-dd">dd</a> <a href="/docs/{{version}}/collections#method-diff">diff</a> <a href="/docs/{{version}}/collections#method-diffkeys">diffKeys</a> <a href="/docs/{{version}}/collections#method-dump">dump</a> <a href="/docs/{{version}}/collections#method-each">each</a> <a href="/docs/{{version}}/collections#method-eachspread">eachSpread</a> <a href="/docs/{{version}}/collections#method-every">every</a> <a href="/docs/{{version}}/collections#method-except">except</a> <a href="/docs/{{version}}/collections#method-filter">filter</a> <a href="/docs/{{version}}/collections#method-first">first</a> <a href="/docs/{{version}}/collections#method-flatmap">flatMap</a> <a href="/docs/{{version}}/collections#method-flatten">flatten</a> <a href="/docs/{{version}}/collections#method-flip">flip</a> <a href="/docs/{{version}}/collections#method-forget">forget</a> <a href="/docs/{{version}}/collections#method-forpage">forPage</a> <a href="/docs/{{version}}/collections#method-get">get</a> <a href="/docs/{{version}}/collections#method-groupby">groupBy</a> <a href="/docs/{{version}}/collections#method-has">has</a> <a href="/docs/{{version}}/collections#method-implode">implode</a> <a href="/docs/{{version}}/collections#method-intersect">intersect</a> <a href="/docs/{{version}}/collections#method-isempty">isEmpty</a> <a href="/docs/{{version}}/collections#method-isnotempty">isNotEmpty</a> <a href="/docs/{{version}}/collections#method-keyby">keyBy</a> <a href="/docs/{{version}}/collections#method-keys">keys</a> <a href="/docs/{{version}}/collections#method-last">last</a> <a href="/docs/{{version}}/collections#method-map">map</a> <a href="/docs/{{version}}/collections#method-mapinto">mapInto</a> <a href="/docs/{{version}}/collections#method-mapspread">mapSpread</a> <a href="/docs/{{version}}/collections#method-maptogroups">mapToGroups</a> <a href="/docs/{{version}}/collections#method-mapwithkeys">mapWithKeys</a> <a href="/docs/{{version}}/collections#method-max">max</a> <a href="/docs/{{version}}/collections#method-median">median</a> <a href="/docs/{{version}}/collections#method-merge">merge</a> <a href="/docs/{{version}}/collections#method-min">min</a> <a href="/docs/{{version}}/collections#method-mode">mode</a> <a href="/docs/{{version}}/collections#method-nth">nth</a> <a href="/docs/{{version}}/collections#method-only">only</a> <a href="/docs/{{version}}/collections#method-pad">pad</a> <a href="/docs/{{version}}/collections#method-partition">partition</a> <a href="/docs/{{version}}/collections#method-pipe">pipe</a> <a href="/docs/{{version}}/collections#method-pluck">pluck</a> <a href="/docs/{{version}}/collections#method-pop">pop</a> <a href="/docs/{{version}}/collections#method-prepend">prepend</a> <a href="/docs/{{version}}/collections#method-pull">pull</a> <a href="/docs/{{version}}/collections#method-push">push</a> <a href="/docs/{{version}}/collections#method-put">put</a> <a href="/docs/{{version}}/collections#method-random">random</a> <a href="/docs/{{version}}/collections#method-reduce">reduce</a> <a href="/docs/{{version}}/collections#method-reject">reject</a> <a href="/docs/{{version}}/collections#method-reverse">reverse</a> <a href="/docs/{{version}}/collections#method-search">search</a> <a href="/docs/{{version}}/collections#method-shift">shift</a> <a href="/docs/{{version}}/collections#method-shuffle">shuffle</a> <a href="/docs/{{version}}/collections#method-slice">slice</a> <a href="/docs/{{version}}/collections#method-sort">sort</a> <a href="/docs/{{version}}/collections#method-sortby">sortBy</a> <a href="/docs/{{version}}/collections#method-sortbydesc">sortByDesc</a> <a href="/docs/{{version}}/collections#method-splice">splice</a> <a href="/docs/{{version}}/collections#method-split">split</a> <a href="/docs/{{version}}/collections#method-sum">sum</a> <a href="/docs/{{version}}/collections#method-take">take</a> <a href="/docs/{{version}}/collections#method-tap">tap</a> <a href="/docs/{{version}}/collections#method-toarray">toArray</a> <a href="/docs/{{version}}/collections#method-tojson">toJson</a> <a href="/docs/{{version}}/collections#method-transform">transform</a> <a href="/docs/{{version}}/collections#method-union">union</a> <a href="/docs/{{version}}/collections#method-unique">unique</a> <a href="/docs/{{version}}/collections#method-uniquestrict">uniqueStrict</a> <a href="/docs/{{version}}/collections#method-unless">unless</a> <a href="/docs/{{version}}/collections#method-values">values</a> <a href="/docs/{{version}}/collections#method-when">when</a> <a href="/docs/{{version}}/collections#method-where">where</a> <a href="/docs/{{version}}/collections#method-wherestrict">whereStrict</a> <a href="/docs/{{version}}/collections#method-wherein">whereIn</a> <a href="/docs/{{version}}/collections#method-whereinstrict">whereInStrict</a> <a href="/docs/{{version}}/collections#method-wherenotin">whereNotIn</a> <a href="/docs/{{version}}/collections#method-wherenotinstrict">whereNotInStrict</a> <a href="/docs/{{version}}/collections#method-zip">zip</a>
  </p>
</div>

<a name="custom-collections"></a>

## Colecciones personalizadas

Para utilizar un objeto `Collection` personalizado con métodos propios, se puede reemplazar el método `newCollection` del modelo:

    <?php
    
    namespace App;
    
    use App\CustomCollection;
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * Create a new Eloquent Collection instance.
         *
         * @param  array  $models
         * @return \Illuminate\Database\Eloquent\Collection
         */
        public function newCollection(array $models = [])
        {
            return new CustomCollection($models);
        }
    }
    

Una vez definido el método `newCollection`, se obtendrá una instancia de esta colección personalizada cada vez que Eloquent retorne una instancia `Collection` de ese modelo. Para utilizar una colección personalizada para cada modelo de la aplicación, debe sobrescribir el método `newCollection` en un modelo base que debe heredarse por el resto de modelos.