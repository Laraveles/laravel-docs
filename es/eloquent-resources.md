# Eloquent: Recursos API

- [Introducción](#introduction)
- [Generating Resources](#generating-resources)
- [Concept Overview](#concept-overview)
- [Writing Resources](#writing-resources) 
    - [Data Wrapping](#data-wrapping)
    - [Pagination](#pagination)
    - [Conditional Attributes](#conditional-attributes)
    - [Conditional Relationships](#conditional-relationships)
    - [Adding Meta Data](#adding-meta-data)
- [Resource Responses](#resource-responses)

<a name="introduction"></a>

## Introducción

Cuando se construye una API, se puede necesitar una capa de transformación que se ubica entre los modelos de Eloquent y las respuestas JSON que se devuelven a los usuarios de la aplicación. Las clases de recursos de Laravel permiten transformar los modelos y las colecciones en JSON.

<a name="generating-resources"></a>

## Generar recursos

Para generar una clase recurso, se puede utilizar el comando de Artisan `make:resource`. Por defecto, los recursos se colocan en el directorio `app/Http/Resources` de la aplicación. Los recursos extienden de la clase `Illuminate\Http\Resources\Json\Resource`:

    php artisan make:resource User
    

#### Recursos para Colecciones

Además de generar recursos que transforman modelos individuales, es posible generar recursos que son responsables de transformar colecciones de modelos. Esto le permite a la respuesta incluir enlaces y otra meta información que es relevante para la colección completa de un recurso en particular.

Para crear un recurso para colecciones, se debe usar la bandera `--collection` cuando se crea el recurso. O, simplemente incluir la palabra `Collection` en el nombre del recurso le indica a Laravel que debe crear un recurso para colecciones. Los recursos para colecciones extienden de la clase `Illuminate\Http\Resources\Json\ResourceCollection`:

    php artisan make:resource Users --collection
    
    php artisan make:resource UserCollection
    

<a name="concept-overview"></a>

## Revisión de conceptos

> {tip} Esta es una presentación de conceptos de alto nivel sobre los recursos y los recursos para colecciones. Se recomienda encarecidamente leer las otras secciones de esta documentación para ganar un conocimiento más profundo sobre la personalización y el poder que ofrecen los recursos.

Antes de entrar en detalle de las opciones disponibles cuando se crean recursos, demos una mirada de alto nivel a cómo se usan los recursos en Laravel. Una clase recurso representa un único modelo que necesita transformarse en una estructura JSON. Por ejemplo, esta es una simple clase de recurso de `User`:

    <?php
    
    namespace App\Http\Resources;
    
    use Illuminate\Http\Resources\Json\Resource;
    
    class User extends Resource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }
    

Cada clase recurso define un método `toArray` que retorna un *array* de atributos que deben ser convertidos a JSON cuando se envíe la respuesta. Tenga en cuenta que se pueden acceder a las propiedades del modelo directamente desde la variable `$this`. Esto es debido a que una clase de recurso automáticamente conectará las propiedades y los métodos del modelo que se inyecta para facilitar el acceso a los mismos. Una vez que se define el recurso, este puede retornarse desde una ruta o un controlador:

    use App\User;
    use App\Http\Resources\User as UserResource;
    
    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });
    

### Colección de recursos

Si se está retornando una colección de recursos o una respuesta paginada, se puede usar el método `collection` cuando se crea la instancia del recurso en la ruta o controlador:

    use App\User;
    use App\Http\Resources\User as UserResource;
    
    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });
    

Por supuesto, esto no permite añadir ninguna ningún meta-dato que pueda ser necesario retornarlo con la colección. Si se quiere personalizar la respuesta del recurso para colecciones, se puede crear un recurso dedicdo a representar la colección:

    php artisan make:resource UserCollection
    

Una vez que se ha generado el recurso para colecciones, se puede definir fácilmente cualquier meta-dato que debería incluirse con la respuesta:

    <?php
    
    namespace App\Http\Resources;
    
    use Illuminate\Http\Resources\Json\ResourceCollection;
    
    class UserCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }
    

Después de definir el recurso para colecciones, se puede retornar desde una ruta o un controlador:

    use App\User;
    use App\Http\Resources\UserCollection;
    
    Route::get('/users', function () {
        return new UserCollection(User::all());
    });
    

<a name="writing-resources"></a>

## Escribir recursos

> {tip} Si no se ha revisado la [revisión de conceptos](#concept-overview), se recomienda encarecidamente hacerlo antes de seguir adelante con la documentación.

Esencialmente, los recursos son simples. Sólo necesitan transformar un modelo dado en *array*. Así, cada recurso contiene un método `toArray` que traduce los atributos de dicho modelo a un *array* que es amigable para la API y que puede ser devuelto a los usuarios:

    <?php
    
    namespace App\Http\Resources;
    
    use Illuminate\Http\Resources\Json\Resource;
    
    class User extends Resource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }
    

Una vez que se define el recurso, puede retornarse directamente desde una ruta o controlador:

    use App\User;
    use App\Http\Resources\User as UserResource;
    
    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });
    

#### Relaciones

Si se quiere incluir un recurso relacionado en la respuesta, se puede simplemente agregarlo en el *array* que se retorna del método `toArray`. En este ejemplo se usa el método `collection` del recurso `Post` para agregar los posts del usuario en la respuesta del recurso:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => Post::collection($this->posts),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
    

> {tip} Si se quiere incluir una relación sólo cuando se haya cargado previamente, revise la documentación sobre [relaciones condicionales](#conditional-relationships).

#### Colección de recursos

Mientras los recursos traducen un modelo en un *array*, los recursos para colecciones traducen una colección de modelos a un *array*. No es absolutamente necesario definir una clase de recurso para colecciones por cada uno de los tipos de modelos existentes ya que todos los recursos incluyen un método `collection` que genera un recurso para colecciones "ad-hoc" en el momento:

    use App\User;
    use App\Http\Resources\User as UserResource;
    
    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });
    

Sin embargo, si se necesita personalizar los meta-datos que se retornan con la colección, será necesario definir un recurso para colecciones:

    <?php
    
    namespace App\Http\Resources;
    
    use Illuminate\Http\Resources\Json\ResourceCollection;
    
    class UserCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }
    

Tal como los recursos singulares, los recursos para colecciones se pueden retornar desde las rutas o los controladores:

    use App\User;
    use App\Http\Resources\UserCollection;
    
    Route::get('/users', function () {
        return new UserCollection(User::all());
    });
    

<a name="data-wrapping"></a>

### Encapsulamiento de datos

Por defecto, el recurso más externo se encapsula en una clave `data` cuando la respuesta se convierte a JSON. Así, por ejemplo, un recurso para colecciones se parece a lo siguiente:

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ]
    }
    

Si se quiere deshabilitar el encapsulamiento del recurso externo, se debe usar el método `withoutWrapping` en la clase base del recurso. Normalmente, se puede llamar a este método desde el `AppServiceProvider` o desde otro [service provider](/docs/{{version}}/providers) que se cargue en cada petición a la aplicación:

    <?php
    
    namespace App\Providers;
    
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Http\Resources\Json\Resource;
    
    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Resource::withoutWrapping();
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
    

> {nota} El método `withoutWrapping` sólo afecta a la respuesta exterior y no removerá las claves `data` que se hayan agregado manualmente a los recursos para colecciones.

### Encapsular recursos anidados

Se tiene total libertad para determinar cómo se encapsulan las relaciones entre los recursos. Si se quiere tener todos los recursos encapsulados dentro de una clave `data`, sin importar su anidado, se debe definir una clase de recurso para colecciones por cada recurso y retornar la colección dentro de una clave `data`.

Por supuesto, queda en duda si esto causará que el recurso exterior sea encapsulados en dos claves `data`. No hay de qué alarmarse, Laravel no permitirá que los recursos se encapsulen dos veces, así que no debe haber preocupación acerca del nivel de anidado de la colección que se está transformando:

    <?php
    
    namespace App\Http\Resources;
    
    use Illuminate\Http\Resources\Json\ResourceCollection;
    
    class CommentsCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return ['data' => $this->collection];
        }
    }
    

### Encapsular datos y paginación

Cuando se retorna una colección paginada en la respuesta, Laravel encapsulará los datos en una clave `data` aunque se haya llamado al método `withoutWrapping`. Esto se hace ya que las respuestas paginadas siempre contienen las claves `meta` y </code>links</0> con información acerca del estado del paginador:

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ],
        "links":{
            "first": "http://example.com/pagination?page=1",
            "last": "http://example.com/pagination?page=1",
            "prev": null,
            "next": null
        },
        "meta":{
            "current_page": 1,
            "from": 1,
            "last_page": 1,
            "path": "http://example.com/pagination",
            "per_page": 15,
            "to": 10,
            "total": 10
        }
    }
    

<a name="pagination"></a>

### Paginación

Siempre se puede pasar una instancia de *paginator* al método `collection` de un recurso o de un recurso para colecciones personalizado:

    use App\User;
    use App\Http\Resources\UserCollection;
    
    Route::get('/users', function () {
        return new UserCollection(User::paginate());
    });
    

Las respuestas paginadas siempre contienen las claves `meta` y `links` con información sobre el estado del *paginator*:

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ],
        "links":{
            "first": "http://example.com/pagination?page=1",
            "last": "http://example.com/pagination?page=1",
            "prev": null,
            "next": null
        },
        "meta":{
            "current_page": 1,
            "from": 1,
            "last_page": 1,
            "path": "http://example.com/pagination",
            "per_page": 15,
            "to": 10,
            "total": 10
        }
    }
    

<a name="conditional-attributes"></a>

### Atributos condicionales

En ocasiones es necesario incluir un atributo en un recurso para colecciones sólo si se cumple con una condición dada. Por ejemplo, se puede querer incluir un valor si el usuario actual es un "administrador". Laravel provee una variedad de métodos auxiliares que son de ayuda en estas situaciones. El método `when` se puede usar para agregar condicionalmente un atributo a una respuesta:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'secret' => $this->when($this->isAdmin(), 'secret-value'),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
    

En este ejemplo, la clave `secret` solo será retornada en la respuesta final si el método `$this->isAdmin()` retorna `true`. Si el método retorna `false`, la clave `secret` se excluirá completamente de la respuesta antes de ser enviada al cliente. El método `when` permite definir los recursos expresivamente sin hacer uso de condicionales durante la construcción del *array*.

El método `when` también acepta un *Closure* como segundo argumento, permitiendo calcular el valor resultante únicamente si la condición dada es `true`:

    'secret' => $this->when($this->isAdmin(), function () {
        return 'secret-value';
    }),
    

> {tip} Recuerde que las llamadas a los métodos en los recursos, se propagan hasta la instancia del modelo subyacente. Así, en este caso, el método `isAdmin` se propaga al modelo Eloquent que se dió originalmente al recurso.

#### Combinar atributos condicionales

En ocasiones, se tienen varios atributos que sólo deben incluirse en la respuesta del recurso si se cumple la misma condición. En ese caso, se puede usar el método `mergeWhen` para incluir los atributos en la respuesta únicamente cuando la condición dada es `true`:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            $this->mergeWhen($this->isAdmin(), [
                'first-secret' => 'value',
                'second-secret' => 'value',
            ]),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
    

De nuevo, si la condición dada es `false`, estos atributos se eliminarán completamente de la respuesta antes de que sea enviada al cliente.

> {nota} El método `mergeWhen` no debería usarse dentro de *arrays* que mezclan claves númericas y de texto. Y más allá, no debería usarse en *arrays* que tienen claves numéricas que no están ordenadas secuencialmente.

<a name="conditional-relationships"></a>

### Relaciones condicionales

Además de cargar atributos de forma condicional, se pueden incluir relaciones condicionales a las respuestas del recurso basadas en si la relación ya se ha cargado en el modelo. Esto le permite al controlador decidir cuáles relaciones deben cargarse en el modelo y el recurso puede fácilmente incluirlas sólo cuando estas han sido cargadas.

Finalmente, esto hace más fácil evitar los problemas de consultas "N+1" en los recursos. El método `whenLoaded` se puede usar para cargar condicionalmente una relación. Para evitar la carga innecesaria de relaciones, este méttodo acepta el nombre de la relación en lugar de la relación propiamente:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => Post::collection($this->whenLoaded('posts')),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
    

En este ejemplo, si la relación no se ha cargado, la clave `posts` se eliminará completamente de la respuesta antes de enviarse al cliente.

#### Información *pivot* condicional

Sumado a incluir condicionalmente información de las relaciones en las respuesta de los recursos, se puede incluir condicionalmente datos de las trablas intermedias en las relaciones muchos-a-muchos usando el método `whenPivotLoaded`. El método `whenPivotLoaded` acepta el nombre de la tabla *pivot* como su primer argumento. El segundo argumento debería ser un *Closure* que defina el valor que debe retornarse si la información está disponible en el modelo:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'expires_at' => $this->whenPivotLoaded('role_users', function () {
                return $this->pivot->expires_at;
            }),
        ];
    }
    

<a name="adding-meta-data"></a>

### Agregar meta-datos

Some JSON API standards require the addition of meta data to your resource and resource collections responses. This often includes things like `links` to the resource or related resources, or meta data about the resource itself. If you need to return additional meta data about a resource, simply include it in your `toArray` method. For example, you might include `link` information when transforming a resource collection:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }
    

When returning additional meta data from your resources, you never have to worry about accidentally overriding the `links` or `meta` keys that are automatically added by Laravel when returning paginated responses. Any additional `links` you define will simply be merged with the links provided by the paginator.

#### Top Level Meta Data

Sometimes you may wish to only include certain meta data with a resource response if the resource is the outer-most resource being returned. Typically, this includes meta information about the response as a whole. To define this meta data, add a `with` method to your resource class. This method should return an array of meta data to be included with the resource response only when the resource is the outer-most resource being rendered:

    <?php
    
    namespace App\Http\Resources;
    
    use Illuminate\Http\Resources\Json\ResourceCollection;
    
    class UserCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return parent::toArray($request);
        }
    
        /**
         * Get additional data that should be returned with the resource array.
         *
         * @param \Illuminate\Http\Request  $request
         * @return array
         */
        public function with($request)
        {
            return [
                'meta' => [
                    'key' => 'value',
                ],
            ];
        }
    }
    

#### Adding Meta Data When Constructing Resources

You may also add top-level data when constructing resource instances in your route or controller. The `additional` method, which is available on all resources, accepts an array of data that should be added to the resource response:

    return (new UserCollection(User::all()->load('roles')))
                    ->additional(['meta' => [
                        'key' => 'value',
                    ]]);
    

<a name="resource-responses"></a>

## Resource Responses

As you have already read, resources may be returned directly from routes and controllers:

    use App\User;
    use App\Http\Resources\User as UserResource;
    
    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });
    

However, sometimes you may need to customize the outgoing HTTP response before it is sent to the client. There are two ways to accomplish this. First, you may chain the `response` method onto the resource. This method will return an `Illuminate\Http\Response` instance, allowing you full control of the response's headers:

    use App\User;
    use App\Http\Resources\User as UserResource;
    
    Route::get('/user', function () {
        return (new UserResource(User::find(1)))
                    ->response()
                    ->header('X-Value', 'True');
    });
    

Alternatively, you may define a `withResponse` method within the resource itself. This method will be called when the resource is returned as the outer-most resource in a response:

    <?php
    
    namespace App\Http\Resources;
    
    use Illuminate\Http\Resources\Json\Resource;
    
    class User extends Resource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
            ];
        }
    
        /**
         * Customize the outgoing response for the resource.
         *
         * @param  \Illuminate\Http\Request
         * @param  \Illuminate\Http\Response
         * @return void
         */
        public function withResponse($request, $response)
        {
            $response->header('X-Value', 'True');
        }
    }