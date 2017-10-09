# Eloquent: Collections

- [Introduction](#introduction)
- [Available Methods](#available-methods)
- [Custom Collections](#custom-collections)

<a name="introduction"></a>

## Introduction

All multi-result sets returned by Eloquent are instances of the `Illuminate\Database\Eloquent\Collection` object, including results retrieved via the `get` method or accessed via a relationship. The Eloquent collection object extends the Laravel [base collection](/docs/{{version}}/collections), so it naturally inherits dozens of methods used to fluently work with the underlying array of Eloquent models.

Of course, all collections also serve as iterators, allowing you to loop over them as if they were simple PHP arrays:

    $users = App\User::where('active', 1)->get();
    
    foreach ($users as $user) {
        echo $user->name;
    }
    

However, collections are much more powerful than arrays and expose a variety of map / reduce operations that may be chained using an intuitive interface. For example, let's remove all inactive models and gather the first name for each remaining user:

    $users = App\User::where('active', 1)->get();
    
    $names = $users->reject(function ($user) {
        return $user->active === false;
    })
    ->map(function ($user) {
        return $user->name;
    });
    

> {note} While most Eloquent collection methods return a new instance of an Eloquent collection, the `pluck`, `keys`, `zip`, `collapse`, `flatten` and `flip` methods return a [base collection](/docs/{{version}}/collections) instance. Likewise, if a `map` operation returns a collection that does not contain any Eloquent models, it will be automatically cast to a base collection.

<a name="available-methods"></a>

## Available Methods

### The Base Collection

All Eloquent collections extend the base [Laravel collection](/docs/{{version}}/collections) object; therefore, they inherit all of the powerful methods provided by the base collection class:

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
    <a href="/docs/{{version}}/collections#method-all">all</a> <a href="/docs/{{version}}/collections#method-average">average</a> <a href="/docs/{{version}}/collections#method-avg">avg</a> <a href="/docs/{{version}}/collections#method-chunk">chunk</a> <a href="/docs/{{version}}/collections#method-collapse">collapse</a> <a href="/docs/{{version}}/collections#method-combine">combine</a> <a href="/docs/{{version}}/collections#method-concat">concat</a> <a href="/docs/{{version}}/collections#method-contains">contains</a> <a href="/docs/{{version}}/collections#method-containsstrict">containsStrict</a> <a href="/docs/{{version}}/collections#method-count">count</a> <a href="/docs/{{version}}/collections#method-crossjoin">crossJoin</a> <a href="/docs/{{version}}/collections#method-dd">dd</a> <a href="/docs/{{version}}/collections#method-diff">diff</a> <a href="/docs/{{version}}/collections#method-diffkeys">diffKeys</a> <a href="/docs/{{version}}/collections#method-dump">dump</a> <a href="/docs/{{version}}/collections#method-each">each</a> <a href="/docs/{{version}}/collections#method-eachspread">eachSpread</a> <a href="/docs/{{version}}/collections#method-every">every</a> <a href="/docs/{{version}}/collections#method-except">except</a> <a href="/docs/{{version}}/collections#method-filter">filter</a> <a href="/docs/{{version}}/collections#method-first">first</a> <a href="/docs/{{version}}/collections#method-flatmap">flatMap</a> <a href="/docs/{{version}}/collections#method-flatten">flatten</a> <a href="/docs/{{version}}/collections#method-flip">flip</a> <a href="/docs/{{version}}/collections#method-forget">forget</a> <a href="/docs/{{version}}/collections#method-forpage">forPage</a> <a href="/docs/{{version}}/collections#method-get">get</a> <a href="/docs/{{version}}/collections#method-groupby">groupBy</a> <a href="/docs/{{version}}/collections#method-has">has</a> <a href="/docs/{{version}}/collections#method-implode">implode</a> <a href="/docs/{{version}}/collections#method-intersect">intersect</a> <a href="/docs/{{version}}/collections#method-isempty">isEmpty</a> <a href="/docs/{{version}}/collections#method-isnotempty">isNotEmpty</a> <a href="/docs/{{version}}/collections#method-keyby">keyBy</a> <a href="/docs/{{version}}/collections#method-keys">keys</a> <a href="/docs/{{version}}/collections#method-last">last</a> <a href="/docs/{{version}}/collections#method-map">map</a> <a href="/docs/{{version}}/collections#method-mapinto">mapInto</a> <a href="/docs/{{version}}/collections#method-mapspread">mapSpread</a> <a href="/docs/{{version}}/collections#method-maptogroups">mapToGroups</a> <a href="/docs/{{version}}/collections#method-mapwithkeys">mapWithKeys</a> <a href="/docs/{{version}}/collections#method-max">max</a> <a href="/docs/{{version}}/collections#method-median">median</a> <a href="/docs/{{version}}/collections#method-merge">merge</a> <a href="/docs/{{version}}/collections#method-min">min</a> <a href="/docs/{{version}}/collections#method-mode">mode</a> <a href="/docs/{{version}}/collections#method-nth">nth</a> <a href="/docs/{{version}}/collections#method-only">only</a> <a href="docs/{{version}}/collections#method-pad">pad</a> <a href="/docs/{{version}}/collections#method-partition">partition</a> <a href="/docs/{{version}}/collections#method-pipe">pipe</a> <a href="/docs/{{version}}/collections#method-pluck">pluck</a> <a href="/docs/{{version}}/collections#method-pop">pop</a> <a href="/docs/{{version}}/collections#method-prepend">prepend</a> <a href="/docs/{{version}}/collections#method-pull">pull</a> <a href="/docs/{{version}}/collections#method-push">push</a> <a href="/docs/{{version}}/collections#method-put">put</a> <a href="/docs/{{version}}/collections#method-random">random</a> <a href="/docs/{{version}}/collections#method-reduce">reduce</a> <a href="/docs/{{version}}/collections#method-reject">reject</a> <a href="/docs/{{version}}/collections#method-reverse">reverse</a> <a href="/docs/{{version}}/collections#method-search">search</a> <a href="/docs/{{version}}/collections#method-shift">shift</a> <a href="/docs/{{version}}/collections#method-shuffle">shuffle</a> <a href="/docs/{{version}}/collections#method-slice">slice</a> <a href="/docs/{{version}}/collections#method-sort">sort</a> <a href="/docs/{{version}}/collections#method-sortby">sortBy</a> <a href="/docs/{{version}}/collections#method-sortbydesc">sortByDesc</a> <a href="/docs/{{version}}/collections#method-splice">splice</a> <a href="/docs/{{version}}/collections#method-split">split</a> <a href="/docs/{{version}}/collections#method-sum">sum</a> <a href="/docs/{{version}}/collections#method-take">take</a> <a href="/docs/{{version}}/collections#method-tap">tap</a> <a href="/docs/{{version}}/collections#method-toarray">toArray</a> <a href="/docs/{{version}}/collections#method-tojson">toJson</a> <a href="/docs/{{version}}/collections#method-transform">transform</a> <a href="/docs/{{version}}/collections#method-union">union</a> <a href="/docs/{{version}}/collections#method-unique">unique</a> <a href="/docs/{{version}}/collections#method-uniquestrict">uniqueStrict</a> <a href="/docs/{{version}}/collections#method-unless">unless</a> <a href="/docs/{{version}}/collections#method-values">values</a> <a href="/docs/{{version}}/collections#method-when">when</a> <a href="/docs/{{version}}/collections#method-where">where</a> <a href="/docs/{{version}}/collections#method-wherestrict">whereStrict</a> <a href="/docs/{{version}}/collections#method-wherein">whereIn</a> <a href="/docs/{{version}}/collections#method-whereinstrict">whereInStrict</a> <a href="/docs/{{version}}/collections#method-wherenotin">whereNotIn</a> <a href="/docs/{{version}}/collections#method-wherenotinstrict">whereNotInStrict</a> <a href="/docs/{{version}}/collections#method-zip">zip</a>
  </p>
</div>

<a name="custom-collections"></a>

## Custom Collections

If you need to use a custom `Collection` object with your own extension methods, you may override the `newCollection` method on your model:

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
    

Once you have defined a `newCollection` method, you will receive an instance of your custom collection anytime Eloquent returns a `Collection` instance of that model. If you would like to use a custom collection for every model in your application, you should override the `newCollection` method on a base model class that is extended by all of your models.