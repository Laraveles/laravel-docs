# Respuestas HTTP

- [Crear respuestas](#creating-responses) 
    - [Añadir cabeceras a las respuestas](#attaching-headers-to-responses)
    - [Añadir *cookies* a las respuestas](#attaching-cookies-to-responses)
    - [*Cookies* & encriptación](#cookies-and-encryption)
- [Redirecciones](#redirects) 
    - [Redirigir a rutas con nombre](#redirecting-named-routes)
    - [Redireccionar a acciones de controladores](#redirecting-controller-actions)
    - [Redireccionar con datos de sesión *flash*](#redirecting-with-flashed-session-data)
- [Otros tipos de respuestas](#other-response-types) 
    - [Respuestas de vistas](#view-responses)
    - [Respuestas JSON](#json-responses)
    - [Descargas de archivos](#file-downloads)
    - [Respuesta de archivos](#file-responses)
- [Macros para respuestas](#response-macros)

<a name="creating-responses"></a>

## Crear respuestas

#### *Strings* & *arrays*

Todas las rutas y controladores deben retornar una respuesta para enviarla al navegador del usuario. Laravel provee diferentes formas para retornar estas respuestas. La respuesta más básica es simplemente devolver una cadena desde una ruta o controlador. El framework convertirá automáticamente la cadena en una respuesta HTTP completa:

    Route::get('/', function () {
        return 'Hello World';
    });
    

Además de devolver cadenas desde sus rutas y controladores, también puede devolver *arrays*. El framework convertirá automáticamente el *array* en una respuesta JSON:

    Route::get('/', function () {
        return [1, 2, 3];
    });
    

> {tip} ¿Sabía que puede retornar [Colecciones Eloquent](/docs/{{version}}/eloquent-collections) desde rutas o controladores? Se convertirán a JSON automáticamente. ¡Pruébelas!

#### Objetos *Response*

Normalmente no solo se retornarán cadenas o *arrays* desde las acciones de las rutas. En su lugar, se retornarán instancias de `Illuminate\Http\Response` o [vistas](/docs/{{version}}/views).

Retornar una instancia `Response` completa permite personalizar el código de respuesta HTTP y sus cabeceras. Una instancia `Response` hereda de la clase `Symfony\Component\HttpFoundation\Response`, la cual provee una gran variedad de métodos para la creación de respuestas HTTP:

    Route::get('home', function () {
        return response('Hello World', 200)
                      ->header('Content-Type', 'text/plain');
    });
    

<a name="attaching-headers-to-responses"></a>

#### Añadir cabeceras a las respuestas

Hay que tener en cuenta que la mayoría de los métodos de una respuesta son encadenables, permitiendo una construcción fluida. Por ejemplo, se puede utilizar el método `header` para añadir una serie de cabeceras a la respuesta antes de devolverla al usuario:

    return response($content)
                ->header('Content-Type', $type)
                ->header('X-Header-One', 'Header Value')
                ->header('X-Header-Two', 'Header Value');
    

O se puede utilizar el método `withHeaders` para especificar un *array* de cabeceras:

    return response($content)
                ->withHeaders([
                    'Content-Type' => $type,
                    'X-Header-One' => 'Header Value',
                    'X-Header-Two' => 'Header Value',
                ]);
    

<a name="attaching-cookies-to-responses"></a>

#### Añadir *cookies* a las respuestas

El método `cookie` en una instancia "Response" permite añadir *cookies* a la misma. Por ejemplo, se puede utilizar el método `cookie` para generar una *cookie* y añadirla a la respuesta de forma fluida:

    return response($content)
                    ->header('Content-Type', $type)
                    ->cookie('name', 'value', $minutes);
    

El método `cookie` acepta además ciertos argumentos que se usan menos frecuentemente. Generalmente estos argumentos tienen el mismo propósito y significado que los argumentos del método nativo de PHP [setcookie](https://secure.php.net/manual/en/function.setcookie.php):

    ->cookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)
    

Por otro lado, se puede utilizar la facade `Cookie` para crear una "cola" de *cookies* para añadir a la respuesta de la aplicación. El método `queue` acepta una instancia de `Cookie` o los argumentos necesarios para crear una instancia de `Cookie`. Éstas *cookies* se adjuntarán a la respuesta antes de que se devuelva al navegador:

    Cookie::queue(Cookie::make('name', 'value', $minutes));
    
    Cookie::queue('name', 'value', $minutes);
    

<a name="cookies-and-encryption"></a>

#### *Cookies* & encriptación

Por defecto, todas las *cookies* generadas por Laravel están encriptadas y firmadas por lo que no pueden ser modificadas o leídas por un cliente. Para desactivar la encriptación de un conjunto de *cookies* generadas por la aplicación, se puede utilizar la propiedad `$except` del middleware `App\Http\Middleware\EncryptCookies`, el cual se encuentra en el directorio `app/Http/Middleware`:

    /**
     * The names of the cookies that should not be encrypted.
     *
     * @var array
     */
    protected $except = [
        'cookie_name',
    ];
    

<a name="redirects"></a>

## Redirecciones

Las redirecciones son instancias de la clase `Illuminate\Http\RedirectResponse` y contienen las cabeceras apropiadas para redirigir al usuario a otra URL. Hay varias formas de generar una instancia de `RedirectResponse`. La más sencilla es utilizando el *helper* global `redirect`:

    Route::get('dashboard', function () {
        return redirect('home/dashboard');
    });
    

En ocasiones puede ser necesario redirigir al usuario a una ubicación anterior, como cuando el envío de un formulario es inválido. Se puede hacer utilizando la función global `back`. Puesto que esta característica utiliza la [sesión](/docs/{{version}}/session), hay que asegurarse de que la ruta que llama al método `back` está usando el grupo de *middleware* `web` o tiene todos los *middleware* de sesión aplicados:

    Route::post('user/profile', function () {
        // Validate the request...
    
        return back()->withInput();
    });
    

<a name="redirecting-named-routes"></a>

### Redireccionar a rutas con nombre

Cuando llama al *helper* `redirect` sin parámetros, se devuelve una instancia de `Illuminate\Routing\Redirector`, lo que le permite llamar a cualquier método en la instancia `Redirector`. Por ejemplo, para generar una respuesta `RedirectResponse` a una ruta determinada, puede utilizar el método `route`:

    return redirect()->route('login');
    

Si su ruta tiene parámetros, puede pasarlos como segundo argumento al método `route`:

    // For a route with the following URI: profile/{id}
    
    return redirect()->route('profile', ['id' => 1]);
    

#### Rellenar parámetros mediante modelos Eloquent

Si está redirigiendo a una ruta con un parámetro "ID" que está siendo traída desde un modelo Eloquent, simplemente puede pasar el modelo mismo. El ID se extraerá automáticamente:

    // For a route with the following URI: profile/{id}
    
    return redirect()->route('profile', [$user]);
    

Si desea personalizar el valor que se coloca en el parámetro de ruta, debe sobreescribir el método `getRouteKey` en su modelo Eloquent:

    /**
     * Get the value of the model's route key.
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }
    

<a name="redirecting-controller-actions"></a>

### Redireccionar a acciones de controladores

También puede generar redirecciones a las [acciones del controlador](/docs/{{version}}/controllers). Para ello, pase el nombre del controlador y de la acción al método `action`. Recuerde que no necesita especificar el *namespace* completo al controlador ya que el `RouteServiceProvider` de Laravel configurará automáticamente el *namespace* del controlador base:

    return redirect()->action('HomeController@index');
    

Si la ruta del controlador requiere parámetros, puede pasarlos como segundo argumento al método `action`:

    return redirect()->action(
        'UserController@profile', ['id' => 1]
    );
    

<a name="redirecting-with-flashed-session-data"></a>

### Redireccionar con datos de sesión *flash*

La redirección a una nueva URL y [flash de los datos a la sesión](/docs/{{version}}/session#flash-data) se hacen generalmente al mismo tiempo. Normalmente, esto se hace después de realizar con éxito una acción como cuando se muestra un mensaje de éxito en la sesión. Para mayor comodidad, puede crear una instancia `RedirectResponse` y los datos *flash* a la sesión en una sola cadena de métodos fluidos:

    Route::post('user/profile', function () {
        // Update the user's profile...
    
        return redirect('dashboard')->with('status', 'Profile updated!');
    });
    

Después de redirigir al usuario, puede mostrar el mensaje de [sesión](/docs/{{version}}/session). Por ejemplo usando [Sintaxis Blade](/docs/{{version}}/blade):

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif
    

<a name="other-response-types"></a>

## Otros tipos de respuestas

El *helper* `response` se puede utilizar para generar otros tipos de instancias de respuesta. Cuando se llama al helper `response` sin argumentos, se devuelve una implementación del [contrato](/docs/{{version}}/contracts) `Illuminate\Contracts\Routing\ResponseFactory`. Este contrato proporciona varios métodos útiles para generar respuestas.

<a name="view-responses"></a>

### Respuestas de vistas

Si necesita control sobre el estado y los encabezados de la respuesta pero también necesita devolver una [vista](/docs/{{version}}/views) como contenido de la respuesta, debe utilizar el método `view`:

    return response()
                ->view('hello', $data, 200)
                ->header('Content-Type', $type);
    

Por supuesto, si no necesita pasar un código de estado HTTP personalizado o encabezados personalizados, debe utilizar el helper global `view`.

<a name="json-responses"></a>

### Respuestas JSON

El método `json` fijará automáticamente el encabezado `Content-Type` a `aplicación/json`, así como convertirá el *array* dado a JSON usando la función PHP `json_encode`:

    return response()->json([
        'name' => 'Abigail',
        'state' => 'CA'
    ]);
    

Si desea crear una respuesta JSONP, puede utilizar el método `json` en combinación con el método `withCallback`:

    return response()
                ->json(['name' => 'Abigail', 'state' => 'CA'])
                ->withCallback($request->input('callback'));
    

<a name="file-downloads"></a>

### Descargas de archivos

El método `download` se puede utilizar para generar una respuesta que obligue al navegador del usuario a descargar el archivo en la ruta dada. El método `download` acepta un nombre de archivo como segundo argumento del método, que determinará el nombre del archivo que verá el usuario que lo descarga. Finalmente, puede pasar un *array* de cabeceras HTTP como tercer argumento al método:

    return response()->download($pathToFile);
    
    return response()->download($pathToFile, $name, $headers);
    
    return response()->download($pathToFile)->deleteFileAfterSend(true);
    

> {note} *Symfony HttpFoundation*, que administra las descargas de archivos, requiere que el archivo descargado tenga un nombre de archivo ASCII.

<a name="file-responses"></a>

### Respuesta de archivos

El método `file` se puede utilizar para mostrar un archivo, como una imagen o PDF, directamente en el navegador del usuario en lugar de iniciar una descarga. Este método acepta la ruta de acceso al archivo como su primer argumento y *array* de cabeceras como su segundo parámetro:

    return response()->file($pathToFile);
    
    return response()->file($pathToFile, $headers);
    

<a name="response-macros"></a>

## Macros para respuestas

Si desea definir una respuesta personalizada que pueda reutilizar en una variedad de rutas y controladores, puede utilizar el método `macro` en la *facade* `Respuesta`. Por ejemplo, desde el método `boot` de un [proveedor de servicios](/docs/{{version}}/providers):

    <?php
    
    namespace App\Providers;
    
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Support\Facades\Response;
    
    class ResponseMacroServiceProvider extends ServiceProvider
    {
        /**
         * Register the application's response macros.
         *
         * @return void
         */
        public function boot()
        {
            Response::macro('caps', function ($value) {
                return Response::make(strtoupper($value));
            });
        }
    }
    

La función `macro` acepta un nombre como primer argumento, y un *Closure* como segundo. El *Closure del macro* se ejecutará cuando se llame al nombre de la macro desde una implementación de `ResponseFactory` o el *helper* `response`:

    return response()->caps('foo');