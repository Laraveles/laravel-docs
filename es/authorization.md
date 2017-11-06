# Autorización

- [Introducción](#introduction)
- [Gates (puertas)](#gates) 
    - [Escribir puertas](#writing-gates)
    - [Autorizar acciones](#authorizing-actions-via-gates)
- [Creación de políticas](#creating-policies) 
    - [Generación de políticas](#generating-policies)
    - [Registro de políticas](#registering-policies)
- [Definición políticas](#writing-policies) 
    - [Métodos de políticas](#policy-methods)
    - [Métodos sin modelos](#methods-without-models)
    - [Filtros de políticas](#policy-filters)
- [Autorizar acciones usando políticas](#authorizing-actions-using-policies) 
    - [Vía el modelo User](#via-the-user-model)
    - [A través de *middleware*](#via-middleware)
    - [A través de *helper* de controladores](#via-controller-helpers)
    - [A través de plantillas Blade](#via-blade-templates)

<a name="introduction"></a>

## Introducción

Además de contar con servicios de [authentication](/docs/{{version}}/authentication) desde la instalación, Laravel también provee una manera simple de autorizar las acciones de los usuarios contra un recurso determinado. Como en la autenticación, el enfoque de Laravel hacia la autorización es simple, y existen principalmente dos maneras de autorizar acciones: *gates* y políticas.

Piense en *gates* y en las políticas como las rutas y los controladores. *Gates* ofrecen una manera simple de autorización basada en funciones anónimas, mientras que las políticas, como los controladores, agrupan la lógica alrededor de un modelo o recurso particular. Primero veremos las *gates* y luego las políticas.

Cuando se construye una aplicación, no es necesario elegir entre usar exclusivamente *gates* o exclusivamente políticas. La mayoría de las aplicaciones probablemente contienen una mezcla de *gates* y políticas ¡y eso es perfectamente correcto! Los *Gates* son más aplicables a las acciones que no están relacionadas con ningun modelo o recurso, tal como ver el dashboard de administración. Por el contrario, las políticas se deberían utilizar cuando se desea autorizar una acción para un modelo o recurso particular.

<a name="gates"></a>

## *Gates*

<a name="writing-gates"></a>

### Escribir *Gates*

Los *Gates* son funciones anónimas que determinan si un usuario está autorizado para realizar una determinada acción y se definen típicamente en la clase `App\Providers\AuthServiceProvider` usando el *facade* `<em>Gate</em>`. Los *Gates* siempre reciben una instancia del usuario como primer argumento, y pueden recibir opcionalmente argumentos adicionales como un modelo Eloquent relevante:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();
    
        Gate::define('update-post', function ($user, $post) {
            return $user->id == $post->user_id;
        });
    }
    

Los *Gates* también se pueden ser definidos usando el estilo de *callback* `Class@method`, como en los controladores:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();
    
        Gate::define('update-post', 'PostPolicy@update');
    }
    

#### *Gates* de recursos

Se pueden definir multiples habilidades del *Gates* usando el método `resource`:

    Gate::resource('posts', 'PostPolicy');
    

Esto es idéntico a definir manualmente las siguientes definiciones de *Gate*:

    Gate::define('posts.view', 'PostPolicy@view');
    Gate::define('posts.create', 'PostPolicy@create');
    Gate::define('posts.update', 'PostPolicy@update');
    Gate::define('posts.delete', 'PostPolicy@delete');
    

Por defecto, se definirán las habilidades `view`, `create`, `update`, y `delete`. Usted puede sobreescribir o agregar a las habilidades por defecto pasando una matriz como tercer argumento al método `resource`. Las claves de la matriz definen los nombres de las habilidades, mientras los valores definen los nombres de los métodos. Por ejemplo, el siguiente código crea las definiciones de dos nuevos *Gates* - `posts.image` y `posts.photo`:

    Gate::resource('posts', 'PostPolicy', [
        'image' => 'updateImage',
        'photo' => 'updatePhoto',
    ]);
    

<a name="authorizing-actions-via-gates"></a>

### Autorizar acciones

Para autorizar acciones usando *Gates*, se deben usar los métodos `allows` o `denies`. Nótese que no es necesario enviar a los métodos el usuario que se encuentra autenticado actualmente. Laravel se encargará automáticamente de pasar el usuario a la función anónima del *Gate*:

    if (Gate::allows('update-post', $post)) {
        // The current user can update the post...
    }
    
    if (Gate::denies('update-post', $post)) {
        // The current user can't update the post...
    }
    

Si se quisiera determinar si un usuario particular está autorizado a realizar una acción, se puede usar el método `forUser` en la *facade* `Gate`:

    if (Gate::forUser($user)->allows('update-post', $post)) {
        // The user can update the post...
    }
    
    if (Gate::forUser($user)->denies('update-post', $post)) {
        // The user can't update the post...
    }
    

<a name="creating-policies"></a>

## Creación de políticas

<a name="generating-policies"></a>

### Generación de políticas

Las Políticas son clases que organizan la lógica de autorización alrededor de un modelo o recurso particular. Por ejemplo, si la aplicación es un blog, se tendría un modelo `Post` y la correspondiente `PostPolicy` para autorizar acciones de los usuarios tales como crear o modificar posts.

Se puede generar una política utilizando el [comando de artisan](/docs/{{version}}/artisan) `make:policy`. La política generada se colocará en el directorio `app/Policies`. Si el directorio no existe en la aplicación, Laravel lo creará:

    php artisan make:policy PostPolicy
    

El comando `make:policy` genera una clase de política vacía. Si se quiere generar una clase con los metodos de políticas básicos de "CRUD" ya incluidos, se puede especificar la bandera `--model` cuando se ejecute el comando:

    php artisan make:policy PostPolicy --model=Post
    

> {tip} Todas las políticas se resuelven via el [service container](/docs/{{version}}/container) de Laravel, lo que permite hacer un *type-hint* de cualquier dependencia necesaria en el constructor de la política e inyectarlos automáticamente.

<a name="registering-policies"></a>

### Registro de políticas

Una vez que la política existe, es necesario que se registre. El `AuthServiceProvider` incluído en las aplicaciones de Laravel contiene una propiedad `policies` la cual mapea los modelos Eloquent con sus políticas correspondientes. Registrar una política le dice a Laravel cuál política utilizar para la autorización de acciones sobre un modelo dado:

    <?php
    
    namespace App\Providers;
    
    use App\Post;
    use App\Policies\PostPolicy;
    use Illuminate\Support\Facades\Gate;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    
    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * The policy mappings for the application.
         *
         * @var array
         */
        protected $policies = [
            Post::class => PostPolicy::class,
        ];
    
        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();
    
            //
        }
    }
    

<a name="writing-policies"></a>

## Definir políticas

<a name="policy-methods"></a>

### Métodos de políticas

Una vez que se ha registrado la política, se pueden incluir métodos para cada una de las acciones que se autorizan. Por ejemplo, se define un método `update` en la política `PostPolicy` la cual determina si un `User` dado puede actualizar una instancia de `Post`.

El método `update` recibirá como argumentos un `User` y una instancia de `Post`, y debería retornar `true` o `false` indicando si el usuario está autorizado a editar el `Post` dado. Así, para este ejemplo, se verifica que el `id` del usuario coincida con el `user_id` del post:

    <?php
    
    namespace App\Policies;
    
    use App\User;
    use App\Post;
    
    class PostPolicy
    {
        /**
         * Determine if the given post can be updated by the user.
         *
         * @param  \App\User  $user
         * @param  \App\Post  $post
         * @return bool
         */
        public function update(User $user, Post $post)
        {
            return $user->id === $post->user_id;
        }
    }
    

Se puede continuar definiendo métodos adicionales en la política a medida que sean necesarios por las distintas acciones a autorizar. Por ejemplo, se podrían definir métodos `view` o `delete` para autorizar las distintas acciones sobre los `Post`, pero hay que recordar que se puede dar cualquier nombre que se desee a los métodos.

> {tip} Si se usa la opción `--model` cuando se crea la política a través de la cónsola de Artisan, esta ya contendrá los métodos para las acciones de `view`, `create`, `update`, y `delete`.

<a name="methods-without-models"></a>

### Métodos sin modelos

Algunos métodos de las políticas recibirán únicamente el usuario autenticado y no una instancia del modelo al cual autorizan. Esta situación es más comun cuando se autorizan acciones de `create`. Por ejemplo, si se está creando un blog, es posible que se quiera verificar si el usuario tiene permisos para crear posts.

Cuando se definen métodos de políticas que no van a recibir una instancia de un modelo, tal como el método `create`, no se recibirá la instancia del modelo. En su lugar, se deben definir métodos que solo esperen el usuario autenticado:

    /**
     * Determine if the given user can create posts.
     *
     * @param  \App\User  $user
     * @return bool
     */
    public function create(User $user)
    {
        //
    }
    

<a name="policy-filters"></a>

### Filtros de políticas

Para ciertos usuarios, se puede querer autorizar todas las acciones dentro de una política dada. Para conseguir esto, se define un método `before` en la política. El método `before` se ejecuta antes de cualquier otro método en la política, dando una oportunidad para autorizar la acción antes de la llamada al método particular. Esta característica se usa más comunmente para autorizar a los administradores de la aplicación a realizar cualquier acción:

    public function before($user, $ability)
    {
        if ($user->isSuperAdmin()) {
            return true;
        }
    }
    

Si se quiere denegar todas las autorizaciones para un usuario, se debe retornar `false` en el método `before`. Si se retorna `null`, la autorización se delegará al método que se ha llamado.

> {nota} El método `before` de una clase política no se ejecuta si la clase no contiene un método con un mobre que coincida con el nombre de la habilidad consultada.

<a name="authorizing-actions-using-policies"></a>

## Autorizar acciones usando políticas

<a name="via-the-user-model"></a>

### Vía el modelo User

El modelo `User` que está incluído en la aplicación Laravel cuenta con dos métodos útiles para autorizar acciones: `can` y `cant`. El método `can` recive la acción que se desea autorizar y el modelo relevante. Por ejemplo, para determinar si un usuario está autorizado a editar un modelo `Post` dado:

    if ($user->can('update', $post)) {
        //
    }
    

Si una [política está registrada](#registering-policies) para el modelo dado, el método `can` llamará automáticamente la política apropiada y devolverá un resultado *boolean*. Si no hay una política registrada para el modelo, el método `can` intentará llamar al Gate y a la función anónima que coincidan con el nombre de la acción dada.

#### Acciones que no requieren modelos

Hay que recordar, que algunas acciones como `create` pueden no requerir una instancia de un modelo. En estas situaciones, se puede pasar el nombre de una clase al método `can`. El nombre de la clase se usará para determinar cuál política utilizar para autorizar la acción:

    use App\Post;
    
    if ($user->can('create', Post::class)) {
        // Executes the "create" method on the relevant policy...
    }
    

<a name="via-middleware"></a>

### Via *middleware*

Laravel incluye un *middleware* que puede autorizar aun antes de que la petición entrante llegue a las rutas o controladores. Por defecto, el *middleware* `Illuminate\Auth\Middleware\Authorize` se le asigna la clave `can` en la clase `App\Http\Kernel`. Exploremos un ejemplo del uso del *middleware* `can` para autorizar que un usuario puede editar un post del blog:

    use App\Post;
    
    Route::put('/post/{post}', function (Post $post) {
        // The current user may update the post...
    })->middleware('can:update,post');
    

En este ejemplo, se están enviando dos argumentos al *middleware* `can`. El primero es el nombre de la acción que se desea autorizar y el segundo es el parámetro en la ruta que se quiere pasar al método de la política. En este caso, dado que se está usando [implicit model binding](/docs/{{version}}/routing#implicit-binding), un modelo `Post` se enviará al método de la política. Si el usuario no está autorizado para realizar la acción dada, una respuesta HTTP con un código de estado `403` será generada por el middleware.

#### Acciones que no requieren modelos

De nuevo, algunas acciones como `create` pueden no requerir una instancia de modelo. En estas situaciones, se puede pasar el nombre de una clase al *middleware*. El nombre de la clase se usará para determinar cuál política se usa para autorizar la acción:

    Route::post('/post', function () {
        // The current user may create posts...
    })->middleware('can:create,App\Post');
    

<a name="via-controller-helpers"></a>

### A través de *helper* de controladores

Adicional a los útiles métodos que provee el modelo `User`, Laravel provee un útil método `authorize` a todos los controladores que extienden de la clase base `App\Http\Controllers\Controller`. Como el método `can`, este método acepta el nombre de la acción que se quiere autorizar y el modelo relevante. Si la acción no está autorizada, el método `authorize` lanzará una `Illuminate\Auth\Access\AuthorizationException`, la cual el manejador de excepciones de Laravel convertirá en una respuesta HTTP con código de estado `403`:

    <?php
    
    namespace App\Http\Controllers;
    
    use App\Post;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;
    
    class PostController extends Controller
    {
        /**
         * Update the given blog post.
         *
         * @param  Request  $request
         * @param  Post  $post
         * @return Response
         */
        public function update(Request $request, Post $post)
        {
            $this->authorize('update', $post);
    
            // The current user can update the blog post...
        }
    }
    

#### Acciones que no requieren modelos

Como se discutió previamente, que algunas acciones como `create` pueden no requerir una instancia de un modelo. En estas situaciones, se puede pasar el nombre de una clase al método `authorize`. El nombre de la clase se usará para determinar cuál política utilizar para autorizar la acción:

    /**
     * Create a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function create(Request $request)
    {
        $this->authorize('create', Post::class);
    
        // The current user can create blog posts...
    }
    

<a name="via-blade-templates"></a>

### A través de plantillas Blade

Cuando se escriben plantillas de Blade, es posible que se quiera mostrar una sección de la página sólo si el usuario está autorizado a ejecutar una acción determinada. Por ejemplo, se puede querer mostrar un formulario de edición para un post del blog sólo si el usuario puede, en efecto, actualizar el post. En esta situación, se puede usar la familia de directivas `@can` y `@cannot`:

    @can('update', $post)
        <!-- The Current User Can Update The Post -->
    @elsecan('create', App\Post::class)
        <!-- The Current User Can Create New Post -->
    @endcan
    
    @cannot('update', $post)
        <!-- The Current User Can't Update The Post -->
    @elsecannot('create', App\Post::class)
        <!-- The Current User Can't Create New Post -->
    @endcannot
    

Estas directivas son atajos convenientes para escribir sentencias `@if` y `@unless`. Las sentencias `@can` y `@cannot` de arriba se traducen respectivamente en las siguientes sentencias:

    @if (Auth::user()->can('update', $post))
        <!-- The Current User Can Update The Post -->
    @endif
    
    @unless (Auth::user()->can('update', $post))
        <!-- The Current User Can't Update The Post -->
    @endunless
    

#### Acciones que no requieren modelos

Como la mayoría de los otros métodos de autorización, se puede pasar el nombre de una clase a las directivas `@can` y `@cannot` si la acción no requiere la instancia de un modelo:

    @can('create', App\Post::class)
        <!-- The Current User Can Create Posts -->
    @endcan
    
    @cannot('create', App\Post::class)
        <!-- The Current User Can't Create Posts -->
    @endcannot