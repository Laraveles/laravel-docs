# Guía de actualización

- [Actualizar a 5.5.0 desde 5.4](#upgrade-5.5.0)

<a name="upgrade-5.5.0"></a>

## Actualizar a 5.5.0 desde 5.4

#### Tiempo estimado de actualización: 1 hora

> {note} Intentamos documentar cada posible cambio de ruptura. Puesto que algunos de estos cambios de ruptura están en partes oscuras del framework, sólo una parte de estos cambios pueden afectar a su aplicación.

### PHP

Laravel 5.5 requiere PHP 7.0.0 o superior.

### Actualización de dependencias

Actualize su dependencia `laravel/framework` a `5.5.*` en el archivo `composer.json`. Además, debe actualizar su dependencia `phpunit/phpunit` a `~6.0`. Luego, agregue el paquete `filp/whoops` con la versión `~2.0` a la sección `require-dev` de su archivo `composer.json`. Por último, en la sección `scripts` de su archivo `composer.json`, añada el comando artisan `package:discover` al evento `post-autoload-dump`:

    "scripts": {
        ...
        "post-autoload-dump": [
            "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
            "@php artisan package:discover"
        ],
    }
    

Por supuesto, no olvide examinar cualquier paquete de terceros consumido por su aplicación y verificar que está usando la versión correcta con soporte para Laravel 5.5.

#### Laravel Installer

> {tip} Si usted usa comúnmente el instalador de Laravel vía `laravel new`, debería actualizar su paquete de instalación de Laravel usando el comando `composer global update`.

#### Laravel Dusk

Laravel Dusk `2.0.0` ha sido liberado para proporcionar compatibilidad con Laravel 5.5 y pruebas de Chrome.

#### Pusher

El *Pusher event broadcasting driver* ahora requiere la versión `~3.0`.

#### Swift Mailer

Laravel 5.5 requiere la versión `~6.0` de *Swift Mailer*.

### Artisan

#### Comandos de Auto-Loading

En Laravel 5.5, el Artisan puede detectar automáticamente los comandos para que no tenga que registrarlos manualmente en su kernel. Para aprovechar esta nueva característica, debe añadir la siguiente línea al método `commands` de su clase `App\Console\Kernel`:

    $this->load(__DIR__.'/Commands');
    

#### El método `fire`

Cualquier método `fire` presente en sus comandos de Artisan debe ser renombrado para `handle`.

#### El comando `optimize`

Con las mejoras recientes en cache de PHP op-code, el comando `optimize` Artisan ya no es necesario. Debe eliminar cualquier referencia a este comando de sus scripts de deploy, ya que se eliminará en una futura versión de Laravel.

### Autorización

#### El método `authorizeResource` de los controladores

Al pasar un nombre de modelo de varias palabras al método `authorizeResource`, el segmento de ruta resultante será "snake case", coincidiendo con el comportamiento de los resource controllers.

#### El método `before` de las políticas (Policies)

El método `before` de una política no se llamará si la clase no contiene un método cuyo nombre coincida con el nombre de la habilidad `(ability)` que se está comprobando.

### Cache

#### Driver de base de datos

Si está usando el driver de caché de base de datos, debe ejecutar `php artisan cache: clear` al desplegar su aplicación Laravel 5.5 actualizada por primera vez.

### Eloquent

#### El método `belongsToMany`

Si está derogando el método `belongsToMany` en su modelo Eloquent, debe actualizar su escritura del método para reflejar la adición de nuevos argumentos:

    /**
     * Define a many-to-many relationship.
     *
     * @param  string  $related
     * @param  string  $table
     * @param  string  $foreignPivotKey
     * @param  string  $relatedPivotKey
     * @param  string  $parentKey
     * @param  string  $relatedKey
     * @param  string  $relation
     * @return \Illuminate\Database\Eloquent\Relations\BelongsToMany
     */
    public function belongsToMany($related, $table = null, $foreignPivotKey = null,
                                  $relatedPivotKey = null, $parentKey = null,
                                  $relatedKey = null, $relation = null)
    {
        //
    }
    

#### BelongsToMany `getQualifiedRelatedKeyName`

El método `getQualifiedRelifiedRelifiedKeyName` ha sido renombrado a `getQualifiedRelatedPivotKeyName`.

#### BelongsToMany `getQualifiedForeignKeyName`

El método `getQualifiedForeignKeyName` ha sido renombrado a `getQualifiedForeignPivotKeyName`.

#### Método `is` en el modelo

Si está sobreescribiendo el método `is` de su modelo Eloquent, se debe eliminar el type-hint `Model` del método. Si está usando el método `is` en su modelo Eloquent, debe eliminar el argumento `$model` del método. Esto permite que el método <0>is</0> reciba nulo como argumento:

    /**
     * Determine if two models have the same ID and belong to the same table.
     *
     * @param  \Illuminate\Database\Eloquent\Model|null  $model
     * @return bool
     */
    public function is($model)
    {
        //
    }
    

#### Propiedad `$events` en el modelo

La propiedad `$events` en tus modelos deben ser renombradas por `$dispatchesEvents`. Este cambio fue hecho porque un numero alto de usuarios necesitan definir una relación `events`, que causó conflicto con el antiguo nombre de la propiedad.

#### Propiedad `$parent` en Pivot

La propiedad protegida `$parent` en la clase `Illuminate\Base\Database\Eloquent\Relations\Pivot` ha sido renombrada a `$pivotParent`.

#### Métodos `create` en las relaciones

Los métodos `BelongsToMany`, `HasOneOrMany`, y `MorphOrMany` `create` de las clases han sido modificados para proporcionar un valor predeterminado para el argumento `$attributes`. Si está usando estos métodos, debería actualizar sus nombres para que coincidan con la nueva definición:

    public function create(array $attributes = [])
    {
        //
    }
    

#### Modelos borrados con "Soft Deleted"

Cuando se utiliza el "soft deleted" con un modelo, la propiedad `exists` en el modelo permanecerá `true`.

#### Formato de columna `withCount`

Cuando se utiliza un alias, el método `withCount` ya no agregará automáticamente `_count` al nombre de la columna resultante. Por ejemplo, en Laravel 5.4, la siguiente consulta daría lugar a que se agregue una columna `bar_count` a la consulta:

    $users = User::withCount('foo as bar')->get();
    

Sin embargo, en Laravel 5.5, el alias se usará exactamente como se da. Si desea añadir `_count` a la columna resultante, debe especificar ese sufijo al definir el alias:

    $users = User::withCount('foo as bar_count')->get();
    

#### Métodos & nombres de atributos del modelo

Para evitar el acceso a las propiedades privadas de un modelo cuando se utiliza el acceso al array, ya no es posible tener un método en el modelo con el mismo nombre que un atributo o propiedad. Si lo hace, se lanzarán excepciones al acceder a los atributos del modelo mediante el acceso al array (`$user['name']`) o el *helper* `data_get`.

### Formato de excepción

En Laravel 5.5, todas las excepciones, incluidas las excepciones de validación, se convierten en respuestas HTTP mediante el gestor de excepciones. Además, el formato por defecto para los errores de validación JSON ha cambiado. El nuevo formato se ajusta a la siguiente convención:

    {
        "message": "The given data was invalid.",
        "errors": {
            "field-1": [
                "Error 1",
                "Error 2"
            ],
            "field-2": [
                "Error 1",
                "Error 2"
            ],
        }
    }
    

Sin embargo, si desea mantener el formato de error Laravel 5.4 JSON, puede agregar el siguiente método a su clase `App\Exceptions\Handler`:

    use Illuminate\Validation\ValidationException;
    
    /**
     * Convert a validation exception into a JSON response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Illuminate\Validation\ValidationException  $exception
     * @return \Illuminate\Http\JsonResponse
     */
    protected function invalidJson($request, ValidationException $exception)
    {
        return response()->json($exception->errors(), $exception->status);
    }
    

#### Intentos de autenticación JSON

Este cambio también afecta al formato del error de validación para los intentos de autenticación realizados sobre JSON. En Laravel 5.5, los fallos de autenticación JSON devolverán los mensajes de error siguiendo la nueva convención de formato descrita anteriormente.

#### Una nota sobre *Form Requests*

Si estaba personalizando el formato de respuesta de una solicitud de formulario individual, ahora debería reemplazar el método `failedValidation` y lanzar una instancia de `HttpResponseException` que contenga su respuesta personalizada:

    use Illuminate\Http\Exceptions\HttpResponseException;
    
    /**
     * Handle a failed validation attempt.
     *
     * @param  \Illuminate\Contracts\Validation\Validator  $validator
     * @return void
     *
     * @throws \Illuminate\Validation\ValidationException
     */
    protected function failedValidation(Validator $validator)
    {
        throw new HttpResponseException(response()->json(..., 422));
    }
    

### Sistema de archivos

#### El método `files`

El método `files` de la clase `Illuminate\Filesystem\Filesystem` ha cambiado para añadir el argumento `$hidden` y ahora devuelve un array de objetos `SplFileInfo`, similar al método `allFiles`. Anteriormente, el método `files` devolvía un array de nombres de ruta. La nueva forma es como sigue:

    public function files($directory, $hidden = false)
    

### Correo

#### Parámetros en desuso

Los argumentos `$data` y `$callback` fueron eliminados de los métodos `queue` y `later` del contrato `Illuminate\Contracts\Mail\MailQueue`:

    /**
     * Queue a new e-mail message for sending.
     *
     * @param  string|array|MailableContract  $view
     * @param  string  $queue
     * @return mixed
     */
    public function queue($view, $queue = null);
    
    /**
     * Queue a new e-mail message for sending after (n) seconds.
     *
     * @param  \DateTimeInterface|\DateInterval|int  $delay
     * @param  string|array|MailableContract  $view
     * @param  string  $queue
     * @return mixed
     */
    public function later($delay, $view, $queue = null);
    

### Colas

#### El helper `dispatch`

Si desea enviar un trabajo que se ejecuta inmediatamente y devuelve un valor del método `handle`, debe utilizar el método `dispatch_now` o `Bus::dispatch`:

    use Illuminate\Support\Facades\Bus;
    
    $value = dispatch_now(new Job);
    
    $value = Bus::dispatchNow(new Job);
    

### Peticiones

#### El método `all`

Si está sobreescribiendo el método `all` de la clase `Illuminate\Http\Request`, debería actualizarlo para reflejar el nuevo argumento `$keys`:

    /**
     * Get all of the input and files for the request.
     *
     * @param  array|mixed  $keys
     * @return array
     */
    public function all($keys = null)
    {
        //
    }
    

#### El método `has`

El método `$request->has` devuelve `true` incluso si el valor de entrada es una cadena vacía o `null`. Se ha añadido un nuevo método `$request->filled` que proporciona el comportamiento anterior del método `has`.

#### El método `intersect`

Se ha eliminado el método `intersect`. Usted puede replicar este comportamiento usando `array_filter` en una llamada a `$request->only`:

    return array_filter($request->only('foo'));
    

#### El método `only`

El método `only` devuelve ahora sólo los atributos que están presentes en la carga útil de petición. Si desea conservar el antiguo comportamiento del método `only`, puede utilizar en su lugar el método `all`.

    return $request->all('foo');
    

#### El helper `request()`

El helper `request` ya no recuperará las claves anidadas. Si es necesario, puede utilizar el método `input` de la petición para lograr este comportamiento:

    return request()->input('filters.date');
    

### Testing

#### Verificación de Autenticación

Algunas verificaciones de autenticación fueron renombradas para una mejor consistencia con el resto de verificación del framework:

<div class="content-list">
  <ul>
    <li>
      <code>seeIsAuthenticated</code> ha sido renombrado como <code>assertAuthenticated</code>.
    </li>
    <li>
      <code>dontSeeIsAuthenticated</code> ha sido renombrado como <code>assertGuest</code>.
    </li>
    <li>
      <code>seeIsAuthenticatedAs</code> ha sido renombrado como <code>assertAuthenticatedAs</code>.
    </li>
    <li>
      <code>seeCredentials</code> ha sido renombrado como <code>assertCredentials</code>.
    </li>
    <li>
      <code>dontSeeCredentials</code> ha sido renombrado como <code>assertInvalidCredentials</code>.
    </li>
  </ul>
</div>

#### Fake Mail

Si está usando el *fake* `Mail` para determinar si un *mailable* fue enviado a la cola (***queued***) durante una solicitud, ahora debería usar `Mail::assertQueued` en lugar de `Mail::assertSent`. Esta distinción le permite afirmar específicamente que el correo se ha puesto en cola para el envío en *background* y no se ha enviado durante la propia solicitud.

#### Tinker

Laravel Tinker ahora soporta omitir namespaces al referirse a las clases de su aplicación. Esta característica requiere un mapa optimizado de la clase Composer, por lo que debe añadir la directiva `optimize-autoloader` a la sección `config` de su archivo `composer.json`:

    "config": {
        ...
        "optimize-autoloader": true
    }
    

### Traducción

#### La `LoaderInterface`

La interfaz `Illuminate\Translation\LoaderInterface` ha sido movida a `Illuminate\Contracts\Translation\Loader`.

### Validación

#### Metodos de validación

Todos los métodos de validación del validador son `public` en lugar de `protected`.

### Vistas

#### Nombre de variables dinámicas con "With"

Al permitir que el método dinámico `__call` comparta variables con una vista, estas variables usarán automáticamente el "camel" case. Por ejemplo, dado lo siguiente:

    return view('pool')->withMaximumVotes(100);
    

La variable `maximumVotes` puede ser accedida de esta manera en la plantilla:

    {{ $maximumVotes }}
    

#### Directiva Blade `@php`

La directiva blade `@php` ya no acepta etiquetas en línea. En su lugar, utilice la forma completa de la directiva:

    @php
        $teamMember = true;
    @endphp
    

### Varios

También le animamos a que vea los cambios en el [Repositorio GitHub](https://github.com/laravel/laravel) `laravel/laravel`. Aunque muchos de estos cambios no son necesarios, es posible que desee mantener estos archivos sincronizados con su aplicación. Algunos de estos cambios se cubrirán en esta guía de actualización, pero otros, como los cambios en los archivos de configuración o los comentarios, no lo serán. Puede ver fácilmente los cambios con la [herramienta de comparación de GitHub](https://github.com/laravel/laravel/compare/5.4...master) y elegir qué actualizaciones le son importantes.