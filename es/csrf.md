# Protección CSRF

- [Introducción](#csrf-introduction)
- [Exluyendo URIs](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>

## Introducción

Laravel hace fácil la protección de su aplicación de ataques[ *cross-site request forgery*](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF). Solicitudes de falsificación a través de sitios *Cross-site request forgeries* son un tipo de ataques maliciosos a través de los cuales comandos no autorizados son ejecutados en nombre de un usuario autenticado.

Laravel genera automáticamente una prueba *token* CSRF para cada sesión de usuario activo administrado por la aplicación. Esta prueba se utiliza para verificar que el usuario autenticado es el que realmente hace las peticiones a la aplicación.

Cada vez que defina un formulario HTML en su aplicación usted debería incluír un campo prueba CSRF oculto en el formulario para que la protección *middleware* pueda validar la peticición. Usted puede utilizar la función de ayuda `csrf_field` para generar el campo con la prueba:

    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>
    

El *middleware* `VerifyCsrfToken` el cuál está incluído en el grupo de *middlewares* web, automaticamente verificará que la prueba en la petición de entrada concuerda con la almacenada en la sesión.

#### Pruebas CSRF y JavasScript

Cudno construye aplicaciones conducidas por JavaScript, es conveniente que su librería JavaScript de HTTP adjunte automáticamente la prueba CSRF a cada petición de salida. Por defecto, el fichero `resources/assets/js/bootstrap.js` registra el valor de la meta etiqueta `prueba csrf` con la librería de HTTP Axios. Si usted no está utilizando esta librería, necesitará configurar manualmente este comportamiento para su aplicación.

<a name="csrf-excluding-uris"></a>

## Excluir URIs de la protección CSRF

A veces puede desear excluír un conjunto de URIs de la protección CSRF. Por ejemplo, si usted está usando [Stripe](https://stripe.com) para procesar pagos y está utilizando su sistema de enganche web, necesitará exluír la ruta del manejador del enganche web de la protección CSRF ya que Stripe no conocerá la prueba CRSF que debe envíar a sus rutas.

Normalmente, usted debería situar este tipo de cosas en rutas fuera de su grupo *middleware* web que el `RouteServiceProvider` aplica a todas las rutas en el fichero `routes/web.php`. No obstante, usted puede también excluír las rutas agregandolas a la propiedad `$except` del *middleware* `VerifyCsrfToken`:

    <?php
    
    namespace App\Http\Middleware;
    
    use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as Middleware;
    
    class VerifyCsrfToken extends Middleware
    {
        /**
         * The URIs that should be excluded from CSRF verification.
         *
         * @var array
         */
        protected $except = [
            'stripe/*',
        ];
    }
    

<a name="csrf-x-csrf-token"></a>

## X-CSRF-TOKEN

Además para comprobar la prueba CSRF como parametro POST el *middleware* `VerifyCsrfToken` comprobará también el `X-CSRF-TOKEN` para las cabeceras de las peticiones. Podría, por ejemplo, almacenar la prueba en una `meta` etiqueta HTML:

    <meta name="csrf-token" content="{{ csrf_token() }}">
    

Entonces, una vez haya creado la `meta` etiqueta, usted puede indicar a una librería como Jquery como agregar automáticamente el token a todas las cabeceras de las peticiones. Esto proporciona una protección CSRF simple y conveniente para sus aplicaciones basadas en AJAX:

    $.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
    });
    

> {tip} Por defecto, el fichero `resources/assets/js/bootstrap.js` registra el valor de la meta etiqueta `crsf-token` con la librería HTTP Axios. Si no está utilizando esta librería, usted necesitará configurar manualmente este comportamiento para su aplicación.

<a name="csrf-x-xsrf-token"></a>

## X-XSRF-TOKEN

Laravel almacena la prueba CSRF en la *cookie* `XSRF-TOKEN` que es incluída con cada respuesta generada por el framework. Puede utilizar la *cookie* para establecer la cabecera de la petición `X-XSRF-TOKEN`.

Esta *cookie* es primordialmente enviada por conveniencia para algunos frameworks de JavaScript u librerías como Angular y Axios, sitúa automáticamente su valor en la cabecera `X-XSRF-TOKEN`.