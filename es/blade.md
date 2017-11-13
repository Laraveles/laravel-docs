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
    - [Procesar vistas para colecciones](#rendering-views-for-collections)
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
    

A veces es útil definir varios *slots* para un componente. Vamos a modificar nuestro componente de alerta para permitir la inyección de un "title". El contenido de los *slots* con nombre se puede mostrar simplemente "imprimiendo" (echoing) la variable que coincide con su nombre:

    <!-- /resources/views/alert.blade.php -->
    
    <div class="alert alert-danger">
        <div class="alert-title">{{ $title }}</div>
    
        {{ $slot }}
    </div>
    

Ahora, podemos inyectar contenido en el *slot* usando la directiva `@slot`. Cualquier contenido que no esté dentro de una directiva `@slot` pasará al componente en la variable `$slot`:

    @component('alert')
        @slot('title')
            Forbidden
        @endslot
    
        You are not allowed to access this resource!
    @endcomponent
    

#### Pasar datos adicionales a los componentes

A veces, es posible que necesite transferir datos adicionales a un componente. Por esta razón, puede pasar una matriz de datos como segundo argumento a la directiva `@componente`. Todos los datos se pondrán a disposición de la plantilla del componente como variables:

    @component('alert', ['foo' => 'bar'])
        ...
    @endcomponent
    

<a name="displaying-data"></a>

## Mostrar datos

Puede mostrar los datos pasados a las vistas de Blade envolviendo la variable entre llaves. Por ejemplo, por la siguiente ruta:

    Route::get('greeting', function () {
        return view('welcome', ['name' => 'Samantha']);
    });
    

Se puede mostrar el contenido de la variable `name` así:

    Hello, {{ $name }}.
    

Por supuesto, el contenido que se puede mostrar no esta limitado a las variables pasadas a la vista. También puede mostrar los resultados de cualquier función PHP. De hecho, poner cualquier código PHP que desee dentro de una declaración de Blade usando *{{}}*:

    The current UNIX timestamp is {{ time() }}.
    

> {tip} Las declaraciones Blade `{{ }}` se envían automáticamente a través de la función PHP `htmlspecialchars` para prevenir ataques XSS.

#### Mostrar datos sin escapar

Por defecto, las sentencias Blade `{{{ }}` se envían automáticamente a través de la función PHP `htmlspecialchars` para prevenir ataques XSS. Para forzar una impresión de datos sin escapar, se puede utilizar la siguiente sintaxis:

    Hello, {!! $name !!}.
    

> {note} Sea muy cuidadoso al hacer muestreo de contenido suministrado por los usuarios de su aplicación. Utilice siempre la sintaxis de doble llave para evitar ataques XSS cuando muestre los datos suministrados por el usuario.

#### Renderizando JSON

A veces puede pasar un *array* a su vista con la intención de renderizarlo como JSON para inicializar una variable JavaScript. Por ejemplo:

    <script>
        var app = <?php echo json_encode($array); ?>;
    </script>
    

Sin embargo, en lugar de llamar manualmente `json_encode`, puede usar la directiva Blade `@json`:

    <script>
        var app = @json($array);
    </script>
    

<a name="blade-and-javascript-frameworks"></a>

### Blade & Frameworks JavaScript

Ya que muchos frameworks de JavaScript también utilizan "llaves" para indicar que una expresión determinada debe mostrarse en el navegador, se puede utilizar el símbolo `@` para informar al motor de procesamiento de Blade que una expresión debe permanecer intacta. Por ejemplo:

    <h1>Laravel</h1>
    
    Hello, @{{ name }}.
    

En este ejemplo, Blade eliminará el símbolo `@`; sin embargo, la expresión `{{ name }}` permanecerá ajena al motor Blade, permitiendo que sea procesada por el framework JavaScript.

#### La directiva `@verbatim`

Si está mostrando variables JavaScript en una gran parte de su plantilla, puede envolver el HTML en la directiva `@verbatim` para que no tenga que prefijar cada expresión con un símbolo `@`:

    @verbatim
        <div class="container">
            Hello, {{ name }}.
        </div>
    @endverbatim
    

<a name="control-structures"></a>

## Estructuras de control

Además de la herencia de plantillas y la visualización de datos, Blade también proporciona accesos directos convenientes para las estructuras de control PHP comunes, tales como sentencias condicionales y bucles. Estos atajos proporcionan una forma muy limpia y concisa de trabajar con las estructuras de control PHP, al mismo tiempo que se mantienen familiarizados con sus contrapartes PHP.

<a name="if-statements"></a>

### Estructuras *if*

Se pueden construir sentencias `if` usando las directivas `@if`, `@elseif`, `@else` y `@endif`. Estas directivas funcionan idénticamente a sus contrapartes de PHP:

    @if (count($records) === 1)
        I have one record!
    @elseif (count($records) > 1)
        I have multiple records!
    @else
        I don't have any records!
    @endif
    

Para conveniencia, Blade ofrece también una directiva `@unless`:

    @unless (Auth::check())
        You are not signed in.
    @endunless
    

Además de las directivas condicionales ya discutidas, las directivas `@isset` y `@empty` pueden ser utilizadas como atajos convenientes para sus respectivas funciones PHP:

    @isset($records)
        // $records is defined and is not null...
    @endisset
    
    @empty($records)
        // $records is "empty"...
    @endempty
    

#### *Shortcuts* (atajos) para autentificacion

Las directivas `@auth` y `@guest` pueden utilizarse para determinar rápidamente si el usuario actual está autenticado o es un invitado:

    @auth
        // The user is authenticated...
    @endauth
    
    @guest
        // The user is not authenticated...
    @endguest
    

Si es necesario, puede especificar el [authentication guard](/docs/{{version}}/authentication) que se debe comprobar al utilizar las directivas `@auth` y `@guest`:

    @auth('admin')
        // The user is authenticated...
    @endauth
    
    @guest('admin')
        // The user is not authenticated...
    @endguest
    

<a name="switch-statements"></a>

### Instrucción *switch*

La instrucción *switch* pueden construirse utilizando las directivas `@switch`, `@case`, `@break`, `@default` y `@endswitch`:

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

### Bucles (*loops*)

Además de las declaraciones condicionales, Blade proporciona directivas simples para trabajar con las estructuras de bucle de PHP. De nuevo, cada una de estas directivas funciona de forma idéntica a sus contrapartes PHP:

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
    

> {tip} Al hacer bucles, puedes usar la variable [loop](#the-loop-variable) para obtener información valiosa sobre el bucle, como si estás en la primera o última iteración.

Al usar bucles, también se puede terminar o saltar la iteración actual:

    @foreach ($users as $user)
        @if ($user->type == 1)
            @continue
        @endif
    
        <li>{{ $user->name }}</li>
    
        @if ($user->number == 5)
            @break
        @endif
    @endforeach
    

También puede incluir la condición con la declaración de directiva en una línea:

    @foreach ($users as $user)
        @continue($user->type == 1)
    
        <li>{{ $user->name }}</li>
    
        @break($user->number == 5)
    @endforeach
    

<a name="the-loop-variable"></a>

### La variable *loop*

Al hacer un bucle, una variable `$loop` estará disponible dentro de el. Esta variable proporciona acceso a algunos datos útiles como el índice actual y si esta es la primer o la última iteración a través del bucle:

    @foreach ($users as $user)
        @if ($loop->first)
            This is the first iteration.
        @endif
    
        @if ($loop->last)
            This is the last iteration.
        @endif
    
        <p>This is user {{ $user->id }}</p>
    @endforeach
    

Si está en un bucle anidado, puede acceder a la variable del bucle padre `$loop` a través de la propiedad `parent`:

    @foreach ($users as $user)
        @foreach ($user->posts as $post)
            @if ($loop->parent->first)
                This is first iteration of the parent loop.
            @endif
        @endforeach
    @endforeach
    

La variable `$loop` también contiene otras propiedades útiles:

| Propiedad             | Descripción                                                      |
| --------------------- | ---------------------------------------------------------------- |
| `$loop->index`     | El índice de la iteración del bucle actual (comienza en 0).      |
| `$loop->iteration` | La iteración del bucle actual (comienza en 1).                   |
| `$loop->remaining` | La iteración que falta en el bucle.                              |
| `$loop->count`     | El número total de elementos en el array que se está iterando.   |
| `$loop->first`     | Si esta es la primera iteración a través del bucle.              |
| `$loop->last`      | Si esta es la última iteración a través del bucle.               |
| `$loop->depth`     | El nivel de anidamiento del bucle actual.                        |
| `$loop->parent`    | Cuando está en un bucle anidado, la variable de bucle del padre. |

<a name="comments"></a>

### Comentarios

Blade también permite definir comentarios en las vistas. Sin embargo, a diferencia de los comentarios HTML, los comentarios Blade no se incluyen en el HTML final de la aplicación:

    {{-- This comment will not be present in the rendered HTML --}}
    

<a name="php"></a>

### PHP

En algunas situaciones, es útil incrustar código PHP en sus vistas. Puede usar la directiva Blade `@php` para ejecutar un bloque de PHP simple dentro de su plantilla:

    @php
        //
    @endphp
    

> {tip} Si bien Blade proporciona esta función, su uso frecuente puede ser una señal de que tiene demasiada lógica incrustada en la plantilla.

<a name="including-sub-views"></a>

## Incluir Sub-Vistas

La directiva `@include` le permite incluir una vista Blade desde otra vista. Todas las variables disponibles en la vista padre se pondrán a disposición de la vista incluida:

    <div>
        @include('shared.errors')
    
        <form>
            <!-- Form Contents -->
        </form>
    </div>
    

Además que la vista incluida heredará todos los datos disponibles en la vista padre, se puede pasar también un *array* de datos a la vista incluida:

    @include('view.name', ['some' => 'data'])
    

Por supuesto, si usted intenta `@include` en una vista que no existe, Laravel lanzará un error. Si desea incluir una vista que puede o no estar presente, debe utilizar la directiva `@includeIf`:

    @includeIf('view.name', ['some' => 'data'])
    

Si se necesita usar `@include` dependiendo de una determinada condición booleana, puede utilizar la directiva `@includeWhen`:

    @includeWhen($boolean, 'view.name', ['some' => 'data'])
    

Para incluir la primera vista que existe desde un conjunto determinado de vistas, puede utilizar la directiva `@includeFirst`:

    @includeFirst(['custom.admin', 'admin'], ['some' => 'data'])
    

> {note} Debe evitar usar las constantes `__DIR__` and `__FILE__` en las vistas Blade, ya que se referirán a la ubicación de la vista compilada en caché.

<a name="rendering-views-for-collections"></a>

### Procesar vistas para colecciones

Se pueden combinar bucles e inclusiones en una línea con la directiva de Blade `@each`:

    @each('view.name', $jobs, 'job')
    

El primer argumento es la vista parcial para representar cada elemento en el *array* o colección. El segundo argumento es el *array* o colección sobre el que se desea iterar, mientras que el tercer argumento es el nombre de la variable que se asignará a la iteración actual dentro de la vista. Así, por ejemplo, si se está iterando sobre un *array* de `jobs`, normalmente se querrá acceder a cada job como una variable `job` en la vista parcial. La clave para la iteración actual estará disponible como variable `key` dentro de su vista parcial.

También se puede pasar un cuarto argumento a la directiva `@each`. Este argumento determina la vista que se mostrará si el *array* está vacío.

    @each('view.name', $jobs, 'job', 'view.empty')
    

> {note} Las vistas mostradas mediante `@each` no heredan las variables de la vista padre. Si la vista hijo requiere estas variables, debe utilizar `@foreach` e `@include`.

<a name="stacks"></a>

## Stacks

Blade permite añadir elementos a *stacks* (pilas) con nombre y que se pueden renderizar en otra vista o *layout*. Esto es especialmente útil para requerir librerías JavaScript en vistas hijas:

    @push('scripts')
        <script src="/example.js"></script>
    @endpush
    

Se pueden añadir tantos elementos al *stack* como se necesite. Para renderizar el contenido completo de un *stack*, únicamente hay que pasar el nombre del mismo a la directiva `@stack`:

    <head>
        <!-- Head Contents -->
    
        @stack('scripts')
    </head>
    

<a name="service-injection"></a>

## Inyección de servicios

La directiva `@inject` puede utilizarse para recuperar un servicio desde el [service container](/docs/{{version}}/container) de Laravel. El primer argumento pasado a `@inject` es el nombre de la variable en la que se ubicará el servicio, mientras que el segundo argumento es el nombre de la clase o interfaz del servicio que desea resolver:

    @inject('metrics', 'App\Services\MetricsService')
    
    <div>
        Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
    </div>
    

<a name="extending-blade"></a>

## Extender Blade

Blade le permite definir sus propias directivas personalizadas utilizando el método `directive`. Cuando el compilador Blade se encuentre con la directiva personalizada, llamará a la llamada de retorno provista con la expresión que contiene la directiva.

El siguiente ejemplo crea una directiva `@datetime ($var)` que formatea una variable `$var`, que debería ser una instancia de `DateTime`:

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
    

Como puede ver, encadenamos el método `format` en cualquier expresión que pase a la directiva. Así, en este ejemplo, el PHP final generado por esta directiva será:

    <?php echo ($var)->format('m/d/Y H:i'); ?>
    

> {note} Después de actualizar la lógica de una directiva Blade, tendrá que borrar todas las vistas almacenadas en caché. Esto se hace utilizando el comando Artisan `view:clear`.

<a name="custom-if-statements"></a>

### Estructuras *if* personalizadas

La programación de una directiva personalizada es a veces más compleja de lo necesario cuando se definen expresiones condicionales simples y personalizadas. Por esta razón, Blade provee un método `Blade::if` que permite definir rápidamente una directiva condicional propia utilizando *Closures*. Por ejemplo, definamos una condición personalizada que compruebe el entorno de aplicación actual. Podemos hacer esto en el método `boot` de nuestro `AppServiceProvider`:

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
    

Una vez que la estructura condicional se ha definido, es muy fácil utilizarla en nuestras plantillas:

    @env('local')
        // The application is in the local environment...
    @elseenv('testing')
        // The application is in the testing environment...
    @else
        // The application is not in the local or testing environment...
    @endenv