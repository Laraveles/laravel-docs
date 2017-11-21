# Upgrade Guide

- [Upgrading To 5.5.0 From 5.4](#upgrade-5.5.0)

<a name="upgrade-5.5.0"></a>

## Upgrading To 5.5.0 From 5.4

#### Estimated Upgrade Time: 1 Hour

> {note} We attempt to document every possible breaking change. Since some of these breaking changes are in obscure parts of the framework only a portion of these changes may actually affect your application.

### PHP

Laravel 5.5 requires PHP 7.0.0 or higher.

### Updating Dependencies

Update your `laravel/framework` dependency to `5.5.*` in your `composer.json` file. In addition, you should update your `phpunit/phpunit` dependency to `~6.0`. Next, add the `filp/whoops` package with version `~2.0` to the `require-dev` section of your `composer.json` file. Finally, in the `scripts` section of your `composer.json` file, add the `package:discover` command to the `post-autoload-dump` event:

    "scripts": {
        ...
        "post-autoload-dump": [
            "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
            "@php artisan package:discover"
        ],
    }
    

Of course, don't forget to examine any 3rd party packages consumed by your application and verify you are using the proper version for Laravel 5.5 support.

#### Laravel Installer

> {tip} If you commonly use the Laravel installer via `laravel new`, you should update your Laravel installer package using the `composer global update` command.

#### Laravel Dusk

Laravel Dusk `2.0.0` has been released to provide compatibility with Laravel 5.5 and headless Chrome testing.

#### Pusher

The Pusher event broadcasting driver now requires version `~3.0` of the Pusher SDK.

#### Swift Mailer

Laravel 5.5 requires version `~6.0` of Swift Mailer.

### Artisan

#### The `fire` Method

Any `fire` methods present on your Artisan commands should be renamed to `handle`.

#### The `optimize` Command

With recent improvements to PHP op-code caching, the `optimize` Artisan command is no longer needed. You should remove any references to this command from your deployment scripts as it will be removed in a future release of Laravel.

### Authorization

#### The `authorizeResource` Controller Method

When passing a multi-word model name to the `authorizeResource` method, the resulting route segment will now be "snake" case, matching the behavior of resource controllers.

#### The `before` Policy Method

The `before` method of a policy class will not be called if the class doesn't contain a method with name matching the name of the ability being checked.

### Cache

#### Database Driver

If you are using the database cache driver, you should run `php artisan cache:clear` when deploying your upgraded Laravel 5.5 application for the first time.

### Eloquent

#### The `belongsToMany` Method

If you are overriding the `belongsToMany` method on your Eloquent model, you should update your method signature to reflect the addition of new arguments:

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

The `getQualifiedRelatedKeyName` method has been renamed to `getQualifiedRelatedPivotKeyName`.

#### BelongsToMany `getQualifiedForeignKeyName`

The `getQualifiedForeignKeyName` method has been renamed to `getQualifiedForeignPivotKeyName`.

#### Model `is` Method

If you are overriding the `is` method of your Eloquent model, you should remove the `Model` type-hint from the method. This allows the `is` method to receive `null` as an argument:

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
    

#### Model `$events` Property

The `$events` property on your models should be renamed to `$dispatchesEvents`. This change was made because of a high number of users needing to define an `events` relationship, which caused a conflict with the old property name.

#### Pivot `$parent` Property

The protected `$parent` property on the `Illuminate\Database\Eloquent\Relations\Pivot` class has been renamed to `$pivotParent`.

#### Relationship `create` Methods

The `BelongsToMany`, `HasOneOrMany`, and `MorphOneOrMany` classes' `create` methods have been modified to provide a default value for the `$attributes` argument. If you are overriding these methods, you should update your signatures to match the new definition:

    public function create(array $attributes = [])
    {
        //
    }
    

#### Soft Deleted Models

When deleting a "soft deleted" model, the `exists` property on the model will remain `true`.

#### `withCount` Column Formatting

When using an alias, the `withCount` method will no longer automatically append `_count` onto the resulting column name. For example, in Laravel 5.4, the following query would result in a `bar_count` column being added to the query:

    $users = User::withCount('foo as bar')->get();
    

However, in Laravel 5.5, the alias will be used exactly as it is given. If you would like to append `_count` to the resulting column, you must specify that suffix when defining the alias:

    $users = User::withCount('foo as bar_count')->get();
    

#### Model Methods & Attribute Names

To prevent accessing a model's private properties when using array access, it's no longer possible to have a model method with the same name as an attribute or property. Doing so will cause exceptions to be thrown when accessing the model's attributes via array access (`$user['name']`) or the `data_get` helper function.

### Exception Format

In Laravel 5.5, all exceptions, including validation exceptions, are converted into HTTP responses by the exception handler. In addition, the default format for JSON validation errors has changed. The new format conforms to the following convention:

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
    

However, if you would like to maintain the Laravel 5.4 JSON error format, you may add the following method to your `App\Exceptions\Handler` class:

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
    

#### JSON Authentication Attempts

This change also affects the validation error formatting for authentication attempts made over JSON. In Laravel 5.5, JSON authentication failures will return the error messages following the new formatting convention described above.

#### A Note On Form Requests

If you were customizing the response format of an individual form request, you should now override the `failedValidation` method of that form request, and throw an `HttpResponseException` instance containing your custom response:

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
    

### Filesystem

#### The `files` Method

The `files` method of the `Illuminate\Filesystem\Filesystem` class has changed its signature to add the `$hidden` argument and now returns an array of `SplFileInfo` objects, similar to the `allFiles` method. Previously, the `files` method returned an array of string path names. The new signature is as follows:

    public function files($directory, $hidden = false)
    

### Mail

#### Unused Parameters

The unused `$data` and `$callback` arguments were removed from the `Illuminate\Contracts\Mail\MailQueue` contract's `queue` and `later` methods:

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
    

### Queues

#### The `dispatch` Helper

If you would like to dispatch a job that runs immediately and returns a value from the `handle` method, you should use the `dispatch_now` or `Bus::dispatchNow` method to dispatch the job:

    use Illuminate\Support\Facades\Bus;
    
    $value = dispatch_now(new Job);
    
    $value = Bus::dispatchNow(new Job);
    

### Requests

#### The `all` Method

If you are overriding the `all` method of the `Illuminate\Http\Request` class, you should update your method signature to reflect the new `$keys` argument:

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
    

#### The `has` Method

The `$request->has` method will now return `true` even if the input value is an empty string or `null`. A new `$request->filled` method has been added that provides the previous behavior of the `has` method.

#### The `intersect` Method

The `intersect` method has been removed. You may replicate this behavior using `array_filter` on a call to `$request->only`:

    return array_filter($request->only('foo'));
    

#### The `only` Method

The `only` method will now only return attributes that are actually present in the request payload. If you would like to preserve the old behavior of the `only` method, you may use the `all` method instead.

    return $request->all('foo');
    

#### The `request()` Helper

The `request` helper will no longer retrieve nested keys. If needed, you may use the `input` method of the request to achieve this behavior:

    return request()->input('filters.date');
    

### Testing

#### Authentication Assertions

Some authentication assertions were renamed for better consistency with the rest of the framework's assertions:

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

#### Mail Fake

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

#### Metodos de validacion

Todos los métodos de validación del validador son `public` en lugar de `protected`.

### Vistas

#### Nombre de variables dinámicas con "With"

When allowing the dynamic `__call` method to share variables with a view, these variables will automatically use "camel" case. For example, given the following:

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