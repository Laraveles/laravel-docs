# Plantillas Blade

- [Introducción](#introduction)
- [Herencia de plantillas](#template-inheritance) 
    - [Definir una plantilla](#defining-a-layout)
    - [Heredar un *layout*](#extending-a-layout)
- [Componentes & Slots](#components-and-slots)
- [Mostrar datos](#displaying-data) 
    - [Blade & Frameworks JavaScript](#blade-and-javascript-frameworks)
- [Estructuras de control](#control-structures) 
    - [Estructuras *if*](#if-statements)
    - [Instrucción *switch*](#switch-statements)
    - [Bucles](#loops)
    - [La variable *loop*](#the-loop-variable)
    - [Comentarios](#comments)
    - [PHP](#php)
- [Incluir sub-vistas](#including-sub-views) 
    - [Rendering Views For Collections](#rendering-views-for-collections)
- [Pilas – *stacks*](#stacks)
- [Inyección de servicios](#service-injection)
- [Extender Blade](#extending-blade) 
    - [Estructuras *if* personalizadas](#custom-if-statements)

<a name="introduction"></a>

## Introducción

Blade es un simple pero poderoso motor de plantillas incluido con Laravel. A diferencia de otros populares motores de plantillas para PHP, Blade no limita el uso de código PHP simple en las vistas. Las vistas en Blade se compilan a código PHP y se cachean hasta que son modificadas, básicamente esto se traduce en que Blade añade sobrecarga cero a las aplicaciones. Las vistas en Blade usan la extensión `.blade.php` y normalmente se almacenan en el directorio `resources/views`.

<a name="template-inheritance"></a>

## Herencia de plantillas

<a name="defining-a-layout"></a>

### Definir una plantilla

Dos de los principales beneficios del uso de Blade son la *herencia de plantillas* y las *secciones*. Para empezar, se va a revisar un sencillo ejemplo. Primero, examinaremos un *layout* "master". Puesto que la mayoría de aplicaciones web mantienen la misma estructura a través de sus diferentes páginas, es conveniente definir este *layout* como una única vista Blade:

    <!-- Stored in resources/views/layouts/app.blade.php -->
    
    <html>
        <head>
            <title>App Name - @yield('title')</title>
        </head>
        <body>
            @section('sidebar')
                This is the master sidebar.
            @show
    
            <div class="container">
                @yield('content')
            </div>
        </body>
    </html>
    

Como se puede observar, este archivo contiene una estructura HTML típica. Sin embargo, se puede tomar nota de las directivas `@section` y `@yield`. La directiva `@section`, como su nombre indica, define un sección de contenido, mientras que la directiva `@yield` es utilizada para mostrar el contenido de una sección.

Una vez que se tiene definido un *layout* para la aplicación, se puede definir una página hija que hereda de este *layout*.

<a name="extending-a-layout"></a>

### Heredar un *layout*

Cuando defina una vista hija, utilice la directiva Blade `@extends` para especificar de qué *layout* debe "heredar". Las vistas que extienden un *layout* de Blade pueden inyectar contenido en las secciones mediante las directivas `@section`. Recordar, como se ve en el ejemplo anterior, el contenido de estas secciones se mostrará el *layout* utilizando `@yield`:

    <!-- Stored in resources/views/child.blade.php -->
    
    @extends('layouts.app')
    
    @section('title', 'Page Title')
    
    @section('sidebar')
        @@parent
    
        <p>This is appended to the master sidebar.</p>
    @endsection
    
    @section('content')
        <p>This is my body content.</p>
    @endsection
    

En este ejemplo, la sección `sidebar` está utilizando la directiva `@parent` para anexar (más que sobrescribir) contenido al sidebar de la plantilla padre. La directiva `@@parent` será reemplazada por el contenido del *layout* cuando se procese la vista.

> {tip} Contrariamente al ejemplo anterior, esta sección `sidebar` termina con `@endsection` en lugar de `@show`. La directiva `@endserction` definirá únicamente una sección mientras que `@show` definirá y **enlazará inmediatamente** la sección.

Las vistas de Blade pueden ser devueltas desde rutas usando el *helper* global `view`:

    Route::get('blade', function () {
        return view('child');
    });
    

<a name="components-and-slots"></a>

## Componentes & Slots

Los componentes y los slots proporcionan beneficios similares a las secciones y los *layouts*; sin embargo, algunos pueden encontrar el modelo mental de componentes y *slots* más fáciles de entender. Primero, imaginemos un componente de "alerta" reutilizable que seria util en toda nuestra aplicación:

    <!-- /resources/views/alert.blade.php -->
    
    <div class="alert alert-danger">
        {{ $slot }}
    </div>
    

La variable `{{ $slot }}` contendrá lo que deseamos inyectar en el componente. Ahora, para construir este componente, podemos usar la directiva Blade `@component`:

    @component('alert')
        <strong>Whoops!</strong> Something went wrong!
    @endcomponent
    

Sometimes it is helpful to define multiple slots for a component. Let's modify our alert component to allow for the injection of a "title". Named slots may be displayed by simply "echoing" the variable that matches their name:

    <!-- /resources/views/alert.blade.php -->
    
    <div class="alert alert-danger">
        <div class="alert-title">{{ $title }}</div>
    
        {{ $slot }}
    </div>
    

Now, we can inject content into the named slot using the `@slot` directive. Any content not within a `@slot` directive will be passed to the component in the `$slot` variable:

    @component('alert')
        @slot('title')
            Forbidden
        @endslot
    
        You are not allowed to access this resource!
    @endcomponent
    

#### Passing Additional Data To Components

Sometimes you may need to pass additional data to a component. For this reason, you can pass an array of data as the second argument to the `@component` directive. All of the data will be made available to the component template as variables:

    @component('alert', ['foo' => 'bar'])
        ...
    @endcomponent
    

<a name="displaying-data"></a>

## Displaying Data

You may display data passed to your Blade views by wrapping the variable in curly braces. For example, given the following route:

    Route::get('greeting', function () {
        return view('welcome', ['name' => 'Samantha']);
    });
    

You may display the contents of the `name` variable like so:

    Hello, {{ $name }}.
    

Of course, you are not limited to displaying the contents of the variables passed to the view. You may also echo the results of any PHP function. In fact, you can put any PHP code you wish inside of a Blade echo statement:

    The current UNIX timestamp is {{ time() }}.
    

> {tip} Blade `{{ }}` statements are automatically sent through PHP's `htmlspecialchars` function to prevent XSS attacks.

#### Displaying Unescaped Data

By default, Blade `{{ }}` statements are automatically sent through PHP's `htmlspecialchars` function to prevent XSS attacks. If you do not want your data to be escaped, you may use the following syntax:

    Hello, {!! $name !!}.
    

> {note} Be very careful when echoing content that is supplied by users of your application. Always use the escaped, double curly brace syntax to prevent XSS attacks when displaying user supplied data.

#### Rendering JSON

Sometimes you may pass an array to your view with the intention of rendering it as JSON in order to initialize a JavaScript variable. For example:

    <script>
        var app = <?php echo json_encode($array); ?>;
    </script>
    

However, instead of manually calling `json_encode`, you may use the `@json` Blade directive:

    <script>
        var app = @json($array);
    </script>
    

<a name="blade-and-javascript-frameworks"></a>

### Blade & JavaScript Frameworks

Since many JavaScript frameworks also use "curly" braces to indicate a given expression should be displayed in the browser, you may use the `@` symbol to inform the Blade rendering engine an expression should remain untouched. For example:

    <h1>Laravel</h1>
    
    Hello, @{{ name }}.
    

In this example, the `@` symbol will be removed by Blade; however, `{{ name }}` expression will remain untouched by the Blade engine, allowing it to instead be rendered by your JavaScript framework.

#### The `@verbatim` Directive

If you are displaying JavaScript variables in a large portion of your template, you may wrap the HTML in the `@verbatim` directive so that you do not have to prefix each Blade echo statement with an `@` symbol:

    @verbatim
        <div class="container">
            Hello, {{ name }}.
        </div>
    @endverbatim
    

<a name="control-structures"></a>

## Control Structures

In addition to template inheritance and displaying data, Blade also provides convenient shortcuts for common PHP control structures, such as conditional statements and loops. These shortcuts provide a very clean, terse way of working with PHP control structures, while also remaining familiar to their PHP counterparts.

<a name="if-statements"></a>

### If Statements

You may construct `if` statements using the `@if`, `@elseif`, `@else`, and `@endif` directives. These directives function identically to their PHP counterparts:

    @if (count($records) === 1)
        I have one record!
    @elseif (count($records) > 1)
        I have multiple records!
    @else
        I don't have any records!
    @endif
    

For convenience, Blade also provides an `@unless` directive:

    @unless (Auth::check())
        You are not signed in.
    @endunless
    

In addition to the conditional directives already discussed, the `@isset` and `@empty` directives may be used as convenient shortcuts for their respective PHP functions:

    @isset($records)
        // $records is defined and is not null...
    @endisset
    
    @empty($records)
        // $records is "empty"...
    @endempty
    

#### Authentication Shortcuts

The `@auth` and `@guest` directives may be used to quickly determine if the current user is authenticated or is a guest:

    @auth
        // The user is authenticated...
    @endauth
    
    @guest
        // The user is not authenticated...
    @endguest
    

If needed, you may specify the [authentication guard](/docs/{{version}}/authentication) that should be checked when using the `@auth` and `@guest` directives:

    @auth('admin')
        // The user is authenticated...
    @endauth
    
    @guest('admin')
        // The user is not authenticated...
    @endguest
    

<a name="switch-statements"></a>

### Switch Statements

Switch statements can be constructed using the `@switch`, `@case`, `@break`, `@default` and `@endswitch` directives:

    @switch($i)
        @case(1)
            First case...
            @break
    
        @case(2)
            Second case...
            @break
    
        @default
            Default case...
    @endswitch
    

<a name="loops"></a>

### Loops

In addition to conditional statements, Blade provides simple directives for working with PHP's loop structures. Again, each of these directives functions identically to their PHP counterparts:

    @for ($i = 0; $i < 10; $i++)
        The current value is {{ $i }}
    @endfor
    
    @foreach ($users as $user)
        <p>This is user {{ $user->id }}</p>
    @endforeach
    
    @forelse ($users as $user)
        <li>{{ $user->name }}</li>
    @empty
        <p>No users</p>
    @endforelse
    
    @while (true)
        <p>I'm looping forever.</p>
    @endwhile
    

> {tip} When looping, you may use the [loop variable](#the-loop-variable) to gain valuable information about the loop, such as whether you are in the first or last iteration through the loop.

When using loops you may also end the loop or skip the current iteration:

    @foreach ($users as $user)
        @if ($user->type == 1)
            @continue
        @endif
    
        <li>{{ $user->name }}</li>
    
        @if ($user->number == 5)
            @break
        @endif
    @endforeach
    

You may also include the condition with the directive declaration in one line:

    @foreach ($users as $user)
        @continue($user->type == 1)
    
        <li>{{ $user->name }}</li>
    
        @break($user->number == 5)
    @endforeach
    

<a name="the-loop-variable"></a>

### The Loop Variable

When looping, a `$loop` variable will be available inside of your loop. This variable provides access to some useful bits of information such as the current loop index and whether this is the first or last iteration through the loop:

    @foreach ($users as $user)
        @if ($loop->first)
            This is the first iteration.
        @endif
    
        @if ($loop->last)
            This is the last iteration.
        @endif
    
        <p>This is user {{ $user->id }}</p>
    @endforeach
    

If you are in a nested loop, you may access the parent loop's `$loop` variable via the `parent` property:

    @foreach ($users as $user)
        @foreach ($user->posts as $post)
            @if ($loop->parent->first)
                This is first iteration of the parent loop.
            @endif
        @endforeach
    @endforeach
    

The `$loop` variable also contains a variety of other useful properties:

| Property              | Description                                            |
| --------------------- | ------------------------------------------------------ |
| `$loop->index`     | The index of the current loop iteration (starts at 0). |
| `$loop->iteration` | The current loop iteration (starts at 1).              |
| `$loop->remaining` | The iteration remaining in the loop.                   |
| `$loop->count`     | The total number of items in the array being iterated. |
| `$loop->first`     | Whether this is the first iteration through the loop.  |
| `$loop->last`      | Whether this is the last iteration through the loop.   |
| `$loop->depth`     | The nesting level of the current loop.                 |
| `$loop->parent`    | When in a nested loop, the parent's loop variable.     |

<a name="comments"></a>

### Comments

Blade also allows you to define comments in your views. However, unlike HTML comments, Blade comments are not included in the HTML returned by your application:

    {{-- This comment will not be present in the rendered HTML --}}
    

<a name="php"></a>

### PHP

In some situations, it's useful to embed PHP code into your views. You can use the Blade `@php` directive to execute a block of plain PHP within your template:

    @php
        //
    @endphp
    

> {tip} While Blade provides this feature, using it frequently may be a signal that you have too much logic embedded within your template.

<a name="including-sub-views"></a>

## Including Sub-Views

Blade's `@include` directive allows you to include a Blade view from within another view. All variables that are available to the parent view will be made available to the included view:

    <div>
        @include('shared.errors')
    
        <form>
            <!-- Form Contents -->
        </form>
    </div>
    

Even though the included view will inherit all data available in the parent view, you may also pass an array of extra data to the included view:

    @include('view.name', ['some' => 'data'])
    

Of course, if you attempt to `@include` a view which does not exist, Laravel will throw an error. If you would like to include a view that may or may not be present, you should use the `@includeIf` directive:

    @includeIf('view.name', ['some' => 'data'])
    

If you would like to `@include` a view depending on a given boolean condition, you may use the `@includeWhen` directive:

    @includeWhen($boolean, 'view.name', ['some' => 'data'])
    

To include the first view that exists from a given array of views, you may use the `includeFirst` directive:

    @includeFirst(['custom.admin', 'admin'], ['some' => 'data'])
    

> {note} You should avoid using the `__DIR__` and `__FILE__` constants in your Blade views, since they will refer to the location of the cached, compiled view.

<a name="rendering-views-for-collections"></a>

### Rendering Views For Collections

You may combine loops and includes into one line with Blade's `@each` directive:

    @each('view.name', $jobs, 'job')
    

The first argument is the view partial to render for each element in the array or collection. The second argument is the array or collection you wish to iterate over, while the third argument is the variable name that will be assigned to the current iteration within the view. So, for example, if you are iterating over an array of `jobs`, typically you will want to access each job as a `job` variable within your view partial. The key for the current iteration will be available as the `key` variable within your view partial.

You may also pass a fourth argument to the `@each` directive. This argument determines the view that will be rendered if the given array is empty.

    @each('view.name', $jobs, 'job', 'view.empty')
    

> {note} Views rendered via `@each` do not inherit the variables from the parent view. If the child view requires these variables, you should use `@foreach` and `@include` instead.

<a name="stacks"></a>

## Stacks

Blade allows you to push to named stacks which can be rendered somewhere else in another view or layout. This can be particularly useful for specifying any JavaScript libraries required by your child views:

    @push('scripts')
        <script src="/example.js"></script>
    @endpush
    

You may push to a stack as many times as needed. To render the complete stack contents, pass the name of the stack to the `@stack` directive:

    <head>
        <!-- Head Contents -->
    
        @stack('scripts')
    </head>
    

<a name="service-injection"></a>

## Service Injection

The `@inject` directive may be used to retrieve a service from the Laravel [service container](/docs/{{version}}/container). The first argument passed to `@inject` is the name of the variable the service will be placed into, while the second argument is the class or interface name of the service you wish to resolve:

    @inject('metrics', 'App\Services\MetricsService')
    
    <div>
        Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
    </div>
    

<a name="extending-blade"></a>

## Extending Blade

Blade allows you to define your own custom directives using the `directive` method. When the Blade compiler encounters the custom directive, it will call the provided callback with the expression that the directive contains.

The following example creates a `@datetime($var)` directive which formats a given `$var`, which should be an instance of `DateTime`:

    <?php
    
    namespace App\Providers;
    
    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;
    
    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Blade::directive('datetime', function ($expression) {
                return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
            });
        }
    
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }
    

As you can see, we will chain the `format` method onto whatever expression is passed into the directive. So, in this example, the final PHP generated by this directive will be:

    <?php echo ($var)->format('m/d/Y H:i'); ?>
    

> {note} After updating the logic of a Blade directive, you will need to delete all of the cached Blade views. The cached Blade views may be removed using the `view:clear` Artisan command.

<a name="custom-if-statements"></a>

### Custom If Statements

Programming a custom directive is sometimes more complex than necessary when defining simple, custom conditional statements. For that reason, Blade provides a `Blade::if` method which allows you to quickly define custom conditional directives using Closures. For example, let's define a custom conditional that checks the current application environment. We may do this in the `boot` method of our `AppServiceProvider`:

    use Illuminate\Support\Facades\Blade;
    
    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('env', function ($environment) {
            return app()->environment($environment);
        });
    }
    

Once the custom conditional has been defined, we can easily use it on our templates:

    @env('local')
        // The application is in the local environment...
    @elseenv('testing')
        // The application is in the testing environment...
    @else
        // The application is not in the local or testing environment...
    @endenv