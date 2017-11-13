# Generación de URL

- [Introducción](#introduction)
- [Conceptos básicos](#the-basics) 
    - [Generar URLs básicas](#generating-basic-urls)
    - [Acceder a la URL actual](#accessing-the-current-url)
- [URL de rutas con nombre](#urls-for-named-routes)
- [URLs a acciones de controladores](#urls-for-controller-actions)
- [Valores por defecto](#default-values)

<a name="introduction"></a>

## Introducción

Laravel incorpora varias funciones de asistencia en la generación de URLs. Por supuesto, son principalmente útiles para la generación de enlaces en las vistas y respuestas API, o para generar redirecciones a otra parte de la aplicación.

<a name="the-basics"></a>

## Conceptos Básicos

<a name="generating-basic-urls"></a>

### Generar URLs básicas

La función `url` se puede utilizar para generar URLs de la aplicación. Estas URLs utilizarán directamente el esquema HTTP o HTTPS y host de la petición:

    $post = App\Post::find(1);
    
    echo url("/posts/{$post->id}");
    
    // http://example.com/posts/1
    

<a name="accessing-the-current-url"></a>

### Acceder a la URL actual

Si no se especifica un directorio a la función `url`, se retornará una instancia de `Illuminate\Routing\UrlGenerator`, permitiendo acceder a información sobre la URL actual:

    // Get the current URL without the query string...
    echo url()->current();
    
    // Get the current URL including the query string...
    echo url()->full();
    
    // Get the full URL for the previous request...
    echo url()->previous();
    

Cada uno de estos métodos se puede acceder también a través de `URL` como [facade](/docs/{{version}}/facades):

    use Illuminate\Support\Facades\URL;
    
    echo URL::current();
    

<a name="urls-for-named-routes"></a>

## URL de rutas con nombre

La función `route` genera URLs a rutas con nombre. Las rutas con nombre permiten generar URLs desacopladas de la URL definida en la ruta. Por lo tanto, si la URL de la ruta cambia, no es necesario modificar las llamadas a la función `route`. Por ejemplo, imaginar una aplicación que contiene la siguiente ruta:

    Route::get('/post/{post}', function () {
        //
    })->name('post.show');
    

Para generar una URL a esta ruta se usaría la función `route` de este modo:

    echo route('post.show', ['post' => 1]);
    
    // http://example.com/post/1
    

A menudo se generarán URLs utilizando la clave primaria de un [modelo Eloquent](/docs/{{version}}/eloquent). Por esta razón, se pueden pasar modelos Eloquent como parámetro. La función `route` extraerá directamente la clave primaria del modelo:

    echo route('post.show', ['post' => $post]);
    

<a name="urls-for-controller-actions"></a>

## URLs a acciones de controladores

La función `action` genera una dirección URL para una acción determinada de un controlador. No se necesita pasar el namespace completo del controlador. En vez de eso, hay que pasar el nombre de la clase controller relativo al namespace `App\Http\Controllers`:

    $url = action('HomeController@index');
    

Si el método del controlador acepta parámetros de ruta, se pueden pasar como segundo argumento a la función:

    $url = action('UserController@profile', ['id' => 1]);
    

<a name="default-values"></a>

## Valores por defecto

Para algunas aplicaciones, se pueden especificar valores por defecto para ciertos parámetros URL. Por ejemplo, imaginar que muchas de las rutas definen un parámetro `{locale}`:

    Route::get('/{locale}/posts', function () {
        //
    })->name('post.index');
    

Es tedioso ir pasando `locale` cada vez que se llama a la función `route`. Se puede utilizar el método `URL::defaults` para definir un valor por defecto para este parámetro que será aplicado siempre durante la petición actual. Se puede llamar a este método desde un [route middleware](/docs/{{version}}/middleware#assigning-middleware-to-routes) (middleware de ruta) por lo que se tendrá acceso a la petición actual:

    <?php
    
    namespace App\Http\Middleware;
    
    use Closure;
    use Illuminate\Support\Facades\URL;
    
    class SetDefaultLocaleForUrls
    {
        public function handle($request, Closure $next)
        {
            URL::defaults(['locale' => $request->user()->locale]);
    
            return $next($request);
        }
    }
    

Una vez que el valor por defecto para `locale` se ha establecido, no será necesario pasarlo más al generar URLs a través del helper `route`.