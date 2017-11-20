# Redirecciones HTTP

- [Crear redirecciones](#creating-redirects)
- [Redireccionar a rutas con nombre](#redirecting-named-routes)
- [Redireccionar a acciones de controladores](#redirecting-controller-actions)
- [Redireccionar con datos de sesión *flash*](#redirecting-with-flashed-session-data)

<a name="creating-redirects"></a>

## Crear redirecciones

Las redirecciones son instancias de la clase `Illuminate\Http\RedirectResponse`, y contienen las cabeceras necesarias redirigir al usuario a otra URL. Hay varias formas de generar una instancia `RedirectResponse`. El método más sencillo es utilizar el *helper* global `redirect`:

    Route::get('dashboard', function () {
        return redirect('home/dashboard');
    });
    

A veces, es posible que desee redirigir al usuario a su ubicación anterior, como cuando un formulario enviado no es válido. Puede hacerlo utilizando el *helper* global `back`. Puesto que esta característica utiliza la [sesión](/docs/{{version}}/session), asegúrese de que la ruta que llama a la función `back` está usando el grupo de *middleware* `web` o tiene todo el *middleware* de sesión aplicado:

    Route::post('user/profile', function () {
        // Validate the request...
    
        return back()->withInput();
    });
    

<a name="redirecting-named-routes"></a>

## Redireccionar a rutas con nombre

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

## Redireccionar a acciones de controladores

También puede generar redirecciones a [acciones de controladores](/docs/{{version}}/controllers). Para ello, pase el nombre del controlador y de la acción al método `action`. Recuerde que no necesita especificar el *namespace* completo al controlador ya que el `RouteServiceProvider` de Laravel configurará automáticamente el espacio de nombres del controlador base:

    return redirect()->action('HomeController@index');
    

Si la ruta del controlador requiere parámetros, puede pasarlos como segundo argumento al método `action`:

    return redirect()->action(
        'UserController@profile', ['id' => 1]
    );
    

<a name="redirecting-with-flashed-session-data"></a>

## Redireccionar con datos de sesión *flash*

La redirección a una nueva URL y [*flash* de los datos a la sesión](/docs/{{version}}/session#flash-data) se hacen generalmente al mismo tiempo. Normalmente, esto se hace después de realizar con éxito una acción como cuando se muestra un mensaje de éxito en la sesión. Para mayor comodidad, puede crear una instancia `RedirectResponse` y los datos *flash* a la sesión en una sola cadena de métodos fluidos:

    Route::post('user/profile', function () {
        // Update the user's profile...
    
        return redirect('dashboard')->with('status', 'Profile updated!');
    });
    

Después de redirigir al usuario, puede mostrar el mensaje de [sesión](/docs/{{version}}/session). Por ejemplo usando [sintaxis Blade](/docs/{{version}}/blade):

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif