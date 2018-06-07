# Eloquent: Relaciones

- [Introducción](#introduction)
- [Definir relaciones](#defining-relationships) 
    - [Uno a uno – *One to one*](#one-to-one)
    - [Uno a muchos – *One to many*](#one-to-many)
    - [Uno a muchos (inverso)](#one-to-many-inverse)
    - [Muchos a muchos – *Many to many*](#many-to-many)
    - [Has Many Through](#has-many-through)
    - [Relaciones polimórficas](#polymorphic-relations)
    - [Relaciones polimórficas muchos a muchos](#many-to-many-polymorphic-relations)
- [Consultas relacionadas](#querying-relations) 
    - [Métodos de relación vs. propiedades dinámicas](#relationship-methods-vs-dynamic-properties)
    - [Consultar la existencia de relaciones](#querying-relationship-existence)
    - [Consultar la inexistencia de relaciones](#querying-relationship-absence)
    - [Contar modelos relacionados](#counting-related-models)
- [Carga temprana – *Eager loading*](#eager-loading) 
    - [Restricciones de carga temprana – *Constraints*](#constraining-eager-loads)
    - [Carga temprana pasiva – *Lazy eager loading*](#lazy-eager-loading)
- [Insertar & actualizar modelos relacionados](#inserting-and-updating-related-models) 
    - [El método `save`](#the-save-method)
    - [El método `create`](#the-create-method)
    - [Relaciones "pertenece a" – *Belongs to*](#updating-belongs-to-relationships)
    - [Relaciones muchos a muchos – *Many to Many*](#updating-many-to-many-relationships)
- [Touching Parent Timestamps](#touching-parent-timestamps)

<a name="introduction"></a>

## Introducción

Las tablas de las bases de datos se relacionan a menudo unas con otras. Por ejemplo, un artículo de un blog puede tener muchos comentarios, o un pedido podría estar relacionado con el usuario que lo solicitó. Eloquent facilita la gestión y el trabajo con estas relaciones fácilmente soportando varios tipos de relaciones diferentes:

- [Uno a uno – *One to one*](#one-to-one)
- [Uno a muchos – *One to many*](#one-to-many)
- [Muchos a muchos – *Many to many*](#many-to-many)
- [Has Many Through](#has-many-through)
- [Relaciones polimórficas](#polymorphic-relations)
- [Relaciones polimórficas muchos a muchos](#many-to-many-polymorphic-relations)

<a name="defining-relationships"></a>

## Definir relaciones

Las relaciones entre modelos Eloquent se definen como métodos en las propias clases. Dado que, como los propios modelos Eloquent, las relaciones también sirven como poderosos [query builders](/docs/{{version}}/queries), la definición de relaciones como métodos proporciona potentes funciones de encadenamiento y consulta de métodos. Por ejemplo, se pueden encadenar restricciones adicionales en esta relación `posts`:

    $user->posts()->where('active', 1)->get();
    

Pero, antes de sumergirnos demasiado en el uso de las relaciones, aprendamos a definir cada tipo.

<a name="one-to-one"></a>

### Uno a uno – *One to one*

La relación uno-a-uno es una relación muy básica. Por ejemplo, un modelo `User` podría estar asociado con uno `Phone`. Para definir esta relación, hay que poner un método `phone` en el modelo `User`. El método `phone` debería llamar al método `hasOne` y retornar su resultado:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * Get the phone record associated with the user.
         */
        public function phone()
        {
            return $this->hasOne('App\Phone');
        }
    }
    

El primer argumento pasado al método `hasOne` es el nombre del modelo relacionado. Una vez que la relación está definida, se deben recuperar los registros relacionados usando las propiedades dinámicas de Eloquent. Las propiedades dinámicas le permiten acceder a los métodos de la relación como si fueran propiedades definidas en el modelo:

    $phone = User::find(1)->phone;
    

Eloquent determina la clave foránea de la relación basada en el nombre del modelo. En este caso, se asume que el modelo `Phone` tiene un clave foránea `user_id`. Si se desea sobrescribir esta convención, hay que pasar un segundo parámetro al método `hasOne`:

    return $this->hasOne('App\Phone', 'foreign_key');
    

Además, Eloquent asume que la llave foránea debe tener un valor que coincida con la columna `id` (o la `$primaryKey` personalizada) columna del padre. En otras palabras, Eloquent buscará el valor de la columna `id` de los usuarios en la columna `user_id` de los registros `Phone`. Si desea que la relación utilice un valor distinto de `id`, puede pasar un tercer parámetro al método `hasOne` especificando la clave personalizada:

    return $this->hasOne('App\Phone', 'foreign_key', 'local_key');
    

#### Definir la inversa de la relación

Por lo tanto, se puede acceder al modelo `Phone` desde `User`. Ahora se definirá una relación en el modelo `Phone` que permitirá acceder al `User` que posee el teléfono. Se puede definir la inversa de la relación `hasOne` utilizando el método `belongsTo`:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Phone extends Model
    {
        /**
         * Get the user that owns the phone.
         */
        public function user()
        {
            return $this->belongsTo('App\User');
        }
    }
    

En el ejemplo anterior, Eloquent intentará emparejar el campo `user_id` del modelo `Phone` con un `id` del modelo `User`. Eloquent nombra la clave ajena por defecto con el nombre del método de la relación y el sufijo `_id`. Sin embargo, si la clave ajena del modelo `Phone` no es `user_id`, se puede pasar una diferente como segundo parámetro el método `belongsTo`:

    /**
     * Get the user that owns the phone.
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key');
    }
    

Si el modelo padre no utiliza `id` como clave primaria, o se desea hacer un join del modelo hijo a una columna diferente, se puede pasar un tercer parámetro al método `belongsTo` especificando la clave personalizada del modelo padre:

    /**
     * Get the user that owns the phone.
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key', 'other_key');
    }
    

<a name="default-models"></a>

#### Modelos por defecto

La relación `belongsTo` permite definir un modelo por defecto para devolver si la relación es `null`. Este patrón se conoce normalmente como [patrón del objeto nulo – *Null Object Pattern*](https://en.wikipedia.org/wiki/Null_Object_pattern) y puede ayudar a eliminar comprobaciones condicionales en el código. El el siguiente ejemplo, la relación `user` devolverá un modelo `App\User` si no hay un `user` asociado al *post*:

    /**
     * Get the author of the post.
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault();
    }
    

Para poblar el modelo por defecto con atributos, se puede pasar un *array* o *Closure* al método `withDefault`:

    /**
     * Get the author of the post.
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault([
            'name' => 'Guest Author',
        ]);
    }
    
    /**
     * Get the author of the post.
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault(function ($user) {
            $user->name = 'Guest Author';
        });
    }
    

<a name="one-to-many"></a>

### Uno a muchos – *One to many*

Una relación "uno-a-muchos" se usa para definir relaciones en las cuales un modelo único posee cualquier cantidad de otros modelos. Por ejemplo, un blog puede tener un número infinito de comentarios. Como en otras relaciones de Eloquent, las relaciones uno-a-muchos se definen colocando una función en el modelo Eloquent:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Post extends Model
    {
        /**
         * Get the comments for the blog post.
         */
        public function comments()
        {
            return $this->hasMany('App\Comment');
        }
    }
    

Hay que recordar que Eloquent determinará automáticamente la columna de la clave externa adecuada en el modelo `Comment`. Por convenio, Eloquent unirá el nombre del propio modelo y el sufijo `_id` mediante la convención "snake_case". Así, para este ejemplo, Eloquent asumirá que la clave ajena del método `Comment` es `post_id`.

Una vez que la relación ha sido definida, se puede acceder a la colección de comentarios accediendo a la propiedad `comments`. Recuerde, puesto que Eloquent provee de "propiedades dinámicas", se puede acceder a los métodos de la relación como si fueran propiedades del modelo:

    $comments = App\Post::find(1)->comments;
    
    foreach ($comments as $comment) {
        //
    }
    

Por supuesto, puesto que todas las relaciones sirven como *query builders*, se pueden agregar agregar *constraints* adicionales a los comentarios que fueron obtenidos con la llamada al método `comments` y continuar la cadena de condiciones en la consulta:

    $comments = App\Post::find(1)->comments()->where('title', 'foo')->first();
    

Como en el método `hasOne`, se puede reemplazar las claves locales y foráneas pasando argumentos adicionales al método `hasMany`:

    return $this->hasMany('App\Comment', 'foreign_key');
    
    return $this->hasMany('App\Comment', 'foreign_key', 'local_key');
    

<a name="one-to-many-inverse"></a>

### Uno a muchos (inverso)

Ahora que podemos acceder a todos los comentarios del *post*, vamos a definir la relación para permitir que un comentario acceda a su publicación padre. Para definir el inverso de una relación `hasMany`, definir una función de relación en el modelo hijo que llame al método `belongsTo`:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Comment extends Model
    {
        /**
         * Get the post that owns the comment.
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }
    

Una vez que la relación se ha definido, se puede recuperar el modelo `Post` desde `Comment` accediendo a la "propiedad dinámica" `post`:

    $comment = App\Comment::find(1);
    
    echo $comment->post->title;
    

En el ejemplo anterior, Eloquent intentará emparejar el campo `user_id` del modelo `Phone` con un `id` del modelo `User`. Eloquent nombra la clave ajena por defecto con el nombre del método de la relación y el sufijo `_id`. Sin embargo, si la clave ajena del modelo `Phone` no es `user_id`, se puede pasar una diferente como segundo parámetro el método `belongsTo`:

    /**
     * Get the post that owns the comment.
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key');
    }
    

Si el modelo padre no utiliza `id` como clave primaria, o se desea hacer un *join* del modelo hijo a una columna diferente, se puede pasar un tercer parámetro al método `belongsTo` especificando la clave personalizada del modelo padre:

    /**
     * Get the post that owns the comment.
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key', 'other_key');
    }
    

<a name="many-to-many"></a>

### Muchos a muchos – *Many to many*

Las relaciones muchos-a-muchos son un poco más complicadas que las `hasOne` o las `hasMany`. Un ejemplo de tal relación es un usuario que contiene varios roles, donde los roles son compartidos por otros usuarios. Por ejemplo, varios usuarios pueden tener el rol de "Admin". Para definir esta relación, se requieren tres tablas de la base de datos: `users`, `roles`, y `role_user`. La tabla `role_user` es derivada del orden alfabético de los nombres de los modelos relacionados y contiene las columnas `user_id` y `role_id`.

Las relaciones muchos-a-muchos se definen con un método que retorna el resultado del método `belongsToMany`. Por ejemplo, definir el método `roles` en el modelo `User`:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * The roles that belong to the user.
         */
        public function roles()
        {
            return $this->belongsToMany('App\Role');
        }
    }
    

Una vez definida la relación, se puede acceder a los roles del usuario usando la propiedad dinámica `roles`:

    $user = App\User::find(1);
    
    foreach ($user->roles as $role) {
        //
    }
    

Por supuesto, como en las otras relaciones, se puede llamar al método `roles` para continuar encadenando restricciones a la consulta sobre la relación:

    $roles = App\User::find(1)->roles()->orderBy('name')->get();
    

Como se ha mencionado anteriormente, para determinar el nombre de la tabla relacionada en un join, Eloquent unirá los nombres de los dos modelos relacionados en orden alfabético. Sin embargo, hay libertad para sobrescribir esta convención. Se puede hacer pasando un segundo parámetro al método `belongsToMany`:

    return $this->belongsToMany('App\Role', 'role_user');
    

Además de poder personalizar el nombre de las tablas en un join, también se puedes personalizar los nombres de las columnas de las claves en la tabla añadiendo más argumentos al método `belongsToMany`. El tercer argumento es el nombre de la clave foránea en la cual se está definiendo la relación, mientras que el cuarto argumento es el nombre de la clave foránea del modelo al que se está haciendo el join:

    return $this->belongsToMany('App\Role', 'role_user', 'user_id', 'role_id');
    

#### Definiendo la Inversa de la Relación

Para definir la inversa de una relación de muchos a muchos, simplemente hay que poner otra llamada a `belongsToMany` en el modelo relacionado. Para continuar con el ejemplo de roles de usuario, se va a definir el método de `users` en el modelo `Role`:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Role extends Model
    {
        /**
         * The users that belong to the role.
         */
        public function users()
        {
            return $this->belongsToMany('App\User');
        }
    }
    

Como se puede ver, la relación está definida exactamente de igual forma que su contraria`User`, con la excepción de que simplemente referencia al modelo `App\User`. Al reutilizar el método `belongsToMany`, todas las opciones para personalizar las claves están disponibles cuando se define la inversa de las relaciones muchos-a-muchos.

#### Recuperar Columnas de Tablas Intermedias

Como se ha visto, trabajar con relaciones muchos-a-muchos requiere la presencia de una tabla intermedia. Eloquent proporciona algunas formas muy útiles de interactuar con esta tabla. Por ejemplo, supongamos que un objeto `User` tiene muchos objetos `Role` con los que se relaciona. Después de acceder a esta relación, se puede acceder a la tabla intermedia utilizando el atributo `pivot` en los modelos:

    $user = App\User::find(1);
    
    foreach ($user->roles as $role) {
        echo $role->pivot->created_at;
    }
    

Hay que tener en cuenta que el modelo obtenido `Role`, está asignado automáticamente a un atributo `pivot`. Este atributo contiene un modelo que representa la tabla intermedia y puede ser utilizado como cualquier otro modelo Eloquent.

Por defecto, sólo las claves del modelo estarán presentes en el objeto `pivot`. Si la tabla pivot contiene atributos adicionales, se deben especificar al definir la relación:

    return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');
    

Si se desea que los campos `crated_at` y `updated_at` se mantengan de forma automática, hay que utilizar el método `withTimestamps` al definir la relación:

    return $this->belongsToMany('App\Role')->withTimestamps();
    

#### Customizing The `pivot` Attribute Name

As noted earlier, attributes from the intermediate table may be accessed on models using the `pivot` attribute. However, you are free to customize the name of this attribute to better reflect its purpose within your application.

For example, if your application contains users that may subscribe to podcasts, you probably have a many-to-many relationship between users and podcasts. If this is the case, you may wish to rename your intermediate table accessor to `subscription` instead of `pivot`. This can be done using the `as` method when defining the relationship:

    return $this->belongsToMany('App\Podcast')
                    ->as('subscription')
                    ->withTimestamps();
    

Once this is done, you may access the intermediate table data using the customized name:

    $users = User::with('podcasts')->get();
    
    foreach ($users->flatMap->podcasts as $podcast) {
        echo $podcast->subscription->created_at;
    }
    

#### Filtering Relationships Via Intermediate Table Columns

You can also filter the results returned by `belongsToMany` using the `wherePivot` and `wherePivotIn` methods when defining the relationship:

    return $this->belongsToMany('App\Role')->wherePivot('approved', 1);
    
    return $this->belongsToMany('App\Role')->wherePivotIn('priority', [1, 2]);
    

#### Defining Custom Intermediate Table Models

If you would like to define a custom model to represent the intermediate table of your relationship, you may call the `using` method when defining the relationship. All custom models used to represent intermediate tables of relationships must extend the `Illuminate\Database\Eloquent\Relations\Pivot` class. For example, we may define a `Role` which uses a custom `UserRole` pivot model:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Role extends Model
    {
        /**
         * The users that belong to the role.
         */
        public function users()
        {
            return $this->belongsToMany('App\User')->using('App\UserRole');
        }
    }
    

When defining the `UserRole` model, we will extend the `Pivot` class:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Relations\Pivot;
    
    class UserRole extends Pivot
    {
        //
    }
    

<a name="has-many-through"></a>

### Has Many Through

The "has-many-through" relationship provides a convenient shortcut for accessing distant relations via an intermediate relation. Por ejemplo, un modelo `Country`, puede tener muchos modelos `Post` a través de un modelo intermedio `User`. En este ejemplo, se podría reunir fácilmente todos los mensajes de un blog para un país determinado. Echemos un vistazo a las tablas para definir esta relación:

    countries
        id - integer
        name - string
    
    users
        id - integer
        country_id - integer
        name - string
    
    posts
        id - integer
        user_id - integer
        title - string
    

Aunque `Post` no contiene una columna `country_id`, la relación `hasManyThrough` proporciona acceso a los mensajes de un país vía `$country->posts`. Para realizar esta consulta, Eloquent inspecciona `country_id` en la tabla intermedia `users`. Después de encontrar los IDs de usuarios coincidentes, serán usados para la consulta a la tabla `posts`.

Ahora que se ha examinado la estructura de la tabla para la relación, se va a definir sobre el modelo `Country`:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Country extends Model
    {
        /**
         * Get all of the posts for the country.
         */
        public function posts()
        {
            return $this->hasManyThrough('App\Post', 'App\User');
        }
    }
    

El primer parámetro pasado al método `hasManyThrough` es el nombre del modelo final al que se desea acceder, mientras que el segundo parámetro es el nombre del modelo intermedio.

Los convenios típicos para claves de Eloquent serán usados para realizar las consultas de la relación. Si se desea personalizar las claves de la relación, se puede hacer por los parámetros tercero y cuarto al método `hasManyThrough`. El tercer parámetro es el nombre de la clave ajena del modelo intermediario. El cuarto parámetro corresponde con el nombre de la clave ajena del modelo final. El quinto argumento es la clave local, mientras que el sexto es la clave local del modelo intermedio:

    class Country extends Model
    {
        public function posts()
        {
            return $this->hasManyThrough(
                'App\Post',
                'App\User',
                'country_id', // Foreign key on users table...
                'user_id', // Foreign key on posts table...
                'id', // Local key on countries table...
                'id' // Local key on users table...
            );
        }
    }
    

<a name="polymorphic-relations"></a>

### Relaciones polimórficas

#### Estructura de tablas

Las relaciones polimorfas permiten a un modelo pertenecer a más de un modelo en una sola asociación. Por ejemplo, imaginar usuarios que pueden "comentar" tanto *posts* como vídeos. Utilizando relaciones polimórficas, se puede utilizar una única tabla `comments` para ambos escenarios. Primero, examinemos la estructura de la tabla requerida para construir esta relación:

    posts
        id - integer
        title - string
        body - text
    
    videos
        id - integer
        title - string
        url - string
    
    comments
        id - integer
        body - text
        commentable_id - integer
        commentable_type - string
    

Las dos columnas a destacar aquí son `commentable_id` y `commentable_type` de la tabla `comments`. La columna `commentable_id` contiene el ID del *post* o *video*, mientras que la columna `commentable_type` contiene el nombre de la clase del modelo propietario. La columna `commentable_type` es como el ORM determina el "tipo" de modelo propietario a retornar cuando se accede a la relación `commentable`.

#### Estructura del modelo

Examinemos que definiciones en el modelo son necesarias para construir esta relación:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Comment extends Model
    {
        /**
         * Get all of the owning commentable models.
         */
        public function commentable()
        {
            return $this->morphTo();
        }
    }
    
    class Post extends Model
    {
        /**
         * Get all of the post's comments.
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }
    
    class Video extends Model
    {
        /**
         * Get all of the video's comments.
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }
    

#### Obtener relaciones polimórficas

Una vez definidas la tabla y los modelos de la base de datos, es posible acceder a las relaciones a través de sus modelos. Por ejemplo, para acceder a los comentarios de un *post*, se puede utilizar la propiedad dinámica `comments</em>:</p>

<pre><code>$post = App\Post::find(1);

foreach ($post->comments as $comment) {
    //
}
`</pre> 

Se puede obtener el propietario de una relación polimórfica desde el modelo polimórfico accediendo al nombre del método que ejecuta la llamada a `morphTo`. En este caso, es el método `commentable` en el modelo `Comment`. Ahora se puede acceder a ese método como propiedad dinámica:

    $comment = App\Comment::find(1);
    
    $commentable = $comment->commentable;
    

La relación `commentable` del modelo `Comment` retornará una instancia de `Post` o `Video`, dependiendo del tipo de modelo al que pertenezca.

#### Tipos polimórficos personalizados

Por defecto, Laravel utilizará el nombre completo de la clase para almacenar el tipo del modelo relacionado. Por ejemplo, dado el ejemplo anterior donde `Comment` puede pertenecer a un `Post` o `Video`, el valor por defecto de `commentable_type` será `App\Post` o `App\Video` respectivamente. Sin embargo, es posible desacoplar la base de datos de la estructura interna de la aplicación. En este caso, se puede definir un "mapa de morfismo" (*morph map*) para instruir a Eloquent en el uso de un nombre personalizado para cada modelo en lugar del nombre de la clase:

    use Illuminate\Database\Eloquent\Relations\Relation;
    
    Relation::morphMap([
        'posts' => 'App\Post',
        'videos' => 'App\Video',
    ]);
    

Se puede registrar el `morphMap` en la función `boot` del `AppServiceProvider` o en un *service provider* separado si se desea.

<a name="many-to-many-polymorphic-relations"></a>

### Relaciones polimórficas muchos a muchos

#### Estructura de Tablas

Además de las relaciones polimórficas tradicionales, se pueden definir relaciones polimórficas "muchos-a-muchos" (many-to-many). Por ejemplo, un `Post` o `Video` de un post podrían compartir una relación polimórfica a un modelo `Tag`. Utilizar relaciones polimórficas muchos-a-muchos permite tener una única lista de tags que se comparten tanto con los blog posts como con los vídeos. Primero, esta sería la estructura de tablas:

    posts
        id - integer
        name - string
    
    videos
        id - integer
        name - string
    
    tags
        id - integer
        name - string
    
    taggables
        tag_id - integer
        taggable_id - integer
        taggable_type - string
    

#### Estructura del Modelo

A continuación, se definen las relaciones en el modelo. Los modelos `Post` y `Video` tendrán un método `tags` que llama al método `morphToMany` de la clase base Eloquent:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Post extends Model
    {
        /**
         * Get all of the tags for the post.
         */
        public function tags()
        {
            return $this->morphToMany('App\Tag', 'taggable');
        }
    }
    

#### Definiendo la Inversa de la Relación

Después, en el modelo `Tag`, se debe definir un método por cada modelo a relacionar. Por lo que, en este ejemplo, se definirá un método `posts` y otro `videos`:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Tag extends Model
    {
        /**
         * Get all of the posts that are assigned this tag.
         */
        public function posts()
        {
            return $this->morphedByMany('App\Post', 'taggable');
        }
    
        /**
         * Get all of the videos that are assigned this tag.
         */
        public function videos()
        {
            return $this->morphedByMany('App\Video', 'taggable');
        }
    }
    

#### Recuperando La Relacion

Una vez definidas la tabla y los modelos de la base de datos, es posible acceder a las relaciones a través de sus modelos. Por ejemplo, para acceder a todas las etiquetas para un post sólo tiene que utilizar la `etiqueta` propiedad dinámica:

    $post = App\Post::find(1);
    
    foreach ($post->tags as $tag) {
        //
    }
    

También puede recuperar el dueño de una relación polimórfica del modelo polimórfico mediante el acceso al nombre del método que realiza la llamada a `morphedByMany`. En nuestro caso, es decir, los `post` o `videos` métodos en la `Etiqueta` modelo. Por lo tanto, usted tendrá acceso a los métodos como propiedades dinámicas:

    $tag = App\Tag::find(1);
    
    foreach ($tag->videos as $video) {
        //
    }
    

<a name="querying-relations"></a>

## Consultas Relacionadas

Since all types of Eloquent relationships are defined via methods, you may call those methods to obtain an instance of the relationship without actually executing the relationship queries. Además, todos los tipos de relaciones Eloquent también sirven como [constructores de consultas](/docs/{{version}}/queries), permitiendo continuar la cadena de restricciones sobre la consulta de la relación antes de finalizar la ejecución de SQL contra la base de datos.

Por ejemplo, imagine un sistema de blog en el cual un `Usuario` modelo tiene muchos `mensajes` modelos asociados:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * Get all of the posts for the user.
         */
        public function posts()
        {
            return $this->hasMany('App\Post');
        }
    }
    

Puede consultar la relación de los `mensajes` y agregar restricciones adicionales a la relación, así como:

    $user = App\User::find(1);
    
    $user->posts()->where('active', 1)->get();
    

Se puede utilizar cualquiera de los métodos de [query builder](/docs/{{version}}/queries) en la relación, por lo que debe asegurarse de explorar la documentación del generador de consultas para obtener información sobre todos los métodos disponibles.

<a name="relationship-methods-vs-dynamic-properties"></a>

### Métodos de Relación Vs. Propiedades Dinámicas

Si no necesita agregar una restricción adicional a una consulta de relación Elocuente, puede simplemente acceder a la relación como si fuese una propiedad. Por ejemplo, al continuar usando nuestro ejemplo de modelos de `usuario` y `mensaje`, necesitaríamos acceso a los mensajes del usuario, así como:

    $user = App\User::find(1);
    
    foreach ($user->posts as $post) {
        //
    }
    

Las propiedades dinámicas son "carga perezosa", significando que sólo cargarán sus datos de relación cuando actualmente se accede a ellos. Por esto, los desarrolladores normalmente usan [carga ansiosa](#eager-loading) para precargar las relaciones que se sabe que serán accedidas después de cargar el modelo. La carga ansiosa provee una significante reducción en las consultas SQL que deben ser ejecutadas para cargar las relaciones de los modelos.

<a name="querying-relationship-existence"></a>

### Consultar la Existencia de Relaciones

Cuando se acceden a los registros de un modelo, se pueden limitar los resultados basados en la existencia de una relación. Por ejemplo, imaginar que se desea obtener todos los posts que contengan al menos un comentario. Para ello, se pasaría el nombre de la relación al método `has` o a `orHas`:

    // Retrieve all posts that have at least one comment...
    $posts = App\Post::has('comments')->get();
    

Además se puede especificar un operador y un contador para personalizar la consulta:

    // Retrieve all posts that have three or more comments...
    $posts = Post::has('comments', '>=', 3)->get();
    

Se pueden anidar estructuras `has` utilizando la notación de "puntos". Por ejemplo, se podrían obtener todos los posts que tienen al menos un comentario y un voto:

    // Retrieve all posts that have at least one comment with votes...
    $posts = Post::has('comments.votes')->get();
    

Si se necesita todavía más control, se pueden utilizar los métodos `whereHas` y `orWhereHas` para incluir condiciones "where" en las consultas `has`. Estos métodos permiten añadir constraints personalizadas a una relación, así como comprobar el contenido de un comentario:

    // Retrieve all posts with at least one comment containing words like foo%
    $posts = Post::whereHas('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();
    

<a name="querying-relationship-absence"></a>

### Consultar la inexistencia de relaciones

Cuando se acceden a los registros de un modelo, se pueden limitar los resultados basados en la inexistencia de una relación. Por ejemplo, imaginar que se desea obtener todos los posts que **no** contengan al menos un comentario. Para ello, se pasaría el nombre de la relación al método `doesntHave` o a `orDoesntHave`:

    $posts = App\Post::doesntHave('comments')->get();
    

Si se necesita todavía más control, se pueden utilizar los métodos `whereDoesntHave` y `orWhereDoesntHave` para incluir condiciones "where" en las consultas `doesntHave`. Estos métodos permiten añadir constraints personalizadas a una relación, así como comprobar el contenido de un comentario:

    $posts = Post::whereDoesntHave('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();
    

<a name="counting-related-models"></a>

### Contar modelos relacionados

If you want to count the number of results from a relationship without actually loading them you may use the `withCount` method, which will place a `{relation}_count` column on your resulting models. Por ejemplo:

    $posts = App\Post::withCount('comments')->get();
    
    foreach ($posts as $post) {
        echo $post->comments_count;
    }
    

You may add the "counts" for multiple relations as well as add constraints to the queries:

    $posts = Post::withCount(['votes', 'comments' => function ($query) {
        $query->where('content', 'like', 'foo%');
    }])->get();
    
    echo $posts[0]->votes_count;
    echo $posts[0]->comments_count;
    

You may also alias the relationship count result, allowing multiple counts on the same relationship:

    $posts = Post::withCount([
        'comments',
        'comments as pending_comments_count' => function ($query) {
            $query->where('approved', false);
        }
    ])->get();
    
    echo $posts[0]->comments_count;
    
    echo $posts[0]->pending_comments_count;
    

<a name="eager-loading"></a>

## Carga temprana

Cuando se acceden a las relaciones de Eloquent como propiedades, los datos de la relación se cargan de forma "diferida" (lazy loaded). Esto quiere decir que los datos de la relación no se cargan hasta que se accede a la propiedad. Sin embargo, Eloquent puede hacer "Carga Ambiciosa" de relaciones en el momento que se consulta al modelo padre. La carga ambiciosa alivia el problema de consulta N + 1. Para ilustrar el problema de consulta N + 1, considere un modelo `Libro` que está relacionado al modelo `Author`:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Book extends Model
    {
        /**
         * Get the author that wrote the book.
         */
        public function author()
        {
            return $this->belongsTo('App\Author');
        }
    }
    

Ahora, vamos a recuperar todos los libros y sus autores:

    $books = App\Book::all();
    
    foreach ($books as $book) {
        echo $book->author->name;
    }
    

Este bucle ejecutará 1 consulta para recuperar todos los libros sobre la mesa, y luego otra consulta por cada libro para recuperar su autor. Por lo tanto, si tenemos 25 libros, este bucle correría 26 consultas: 1 por el libro original, y 25 consultas adicionales para recuperar el autor de cada libro.

Afortunadamente, podemos usar la carga ambiciosa para reducir esta operación a 2 consultas. Cuando se consulta, tu puedes especificar cuales relaciones deben cargarse también usando el método `with`:

    $books = App\Book::with('author')->get();
    
    foreach ($books as $book) {
        echo $book->author->name;
    }
    

Para esta operación, únicamente se ejecutarán dos consultas:

    select * from books
    
    select * from authors where id in (1, 2, 3, 4, 5, ...)
    

#### Carga Ambiciosa De Relaciones Múltiples

A veces puede necesitar hacer carga ambiciosa de diferentes relaciones en una sola operación. Para ello, solo tiene que pasar argumentos adicionales al método `with`:

    $books = App\Book::with(['author', 'publisher'])->get();
    

#### Carga temprana anidada

Para hacer Carga Ambiciosa de relaciones anidadas, debes usar la sintaxis "punto". Por Ejemplo, hagamos Carga Ambiciosa de todos los autores de los libros y de todos los contactos personales de los autores en una sentencia de Eloquent:

    $books = App\Book::with('author.contacts')->get();
    

#### Eager Loading Specific Columns

You may not always need every column from the relationships you are retrieving. For this reason, Eloquent allows you to specify which columns of the relationship you would like to retrieve:

    $users = App\Book::with('author:id,name')->get();
    

> {note} When using this feature, you should always include the `id` column in the list of columns you wish to retrieve.

<a name="constraining-eager-loads"></a>

### Restringiendo la carga temprana

A veces puedes desear hacer Carga Ambiciosa a una relación, pero también especificar restricciones adicionales a la consulta para la hacer la Carga Ambiciosa. He aquí un ejemplo:

    $users = App\User::with(['posts' => function ($query) {
        $query->where('title', 'like', '%first%');
    }])->get();
    

In this example, Eloquent will only eager load posts where the post's `title` column contains the word `first`. Of course, you may call other [query builder](/docs/{{version}}/queries) methods to further customize the eager loading operation:

    $users = App\User::with(['posts' => function ($query) {
        $query->orderBy('created_at', 'desc');
    }])->get();
    

<a name="lazy-eager-loading"></a>

### Carga anticipada (Lazy Eager Loading)

A veces puede que tenga que hacer Carga Ambiciosa de una relación después de que el modelo padre ya se ha recuperado. Por ejemplo, esto puede ser útil si usted necesita decidir dinámicamente si debe cargar modelos relacionados:

    $books = App\Book::all();
    
    if ($someCondition) {
        $books->load('author', 'publisher');
    }
    

If you need to set additional query constraints on the eager loading query, you may pass an array keyed by the relationships you wish to load. The array values should be `Closure` instances which receive the query instance:

    $books->load(['author' => function ($query) {
        $query->orderBy('published_date', 'asc');
    }]);
    

To load a relationship only when it has not already been loaded, use the `loadMissing` method:

    public function format(Book $book)
    {
        $book->loadMissing('author');
    
        return [
            'name' => $book->name,
            'author' => $book->author->name
        ];
    }
    

<a name="inserting-and-updating-related-models"></a>

## Inserting & Updating Related Models

<a name="the-save-method"></a>

### El método Save

Eloquent provee métodos convenientes para la adición de nuevos modelos a las relaciones. Por ejemplo, quizás necesite insertar un nuevo `Comment` a un modelo `Post`. En lugar de configurar manualmente el atributo `post_id` en el `Comment`, puede insertar el `Comment` directamente desde el método `save` de la relación:

    $comment = new App\Comment(['message' => 'A new comment.']);
    
    $post = App\Post::find(1);
    
    $post->comments()->save($comment);
    

Nótese que no accedemos a los `comments` de la relación como una propiedad dinámica. En su lugar, llamamos al método `comments` para obtener una instancia de la relación. El método `save` agregará automáticamente el valor `post_id` apropiado al nuevo modelo `Comment`.

Si usted necesita grabar multiples modelos relacionados, puede usar el método **saveMany**

    $post = App\Post::find(1);
    
    $post->comments()->saveMany([
        new App\Comment(['message' => 'A new comment.']),
        new App\Comment(['message' => 'Another comment.']),
    ]);
    

<a name="the-create-method"></a>

### El método Create

Además de los métodos `save` y `saveMany`, usted puede utilizar también el método `create`, el cuál acepta una matriz de atributos, crea el modelo y lo inserta en la base de datos De nuevo, la diferencia entre `save` y `create` es que `save` acepta una instancia de un modelo completo de Eloquent mientras que `create` acepta una `matriz` de PHP:

    $post = App\Post::find(1);
    
    $comment = $post->comments()->create([
        'message' => 'A new comment.',
    ]);
    

> {tip} Before using the `create` method, be sure to review the documentation on attribute [mass assignment](/docs/{{version}}/eloquent#mass-assignment).

You may use the `createMany` method to create multiple related models:

    $post = App\Post::find(1);
    
    $post->comments()->createMany([
        [
            'message' => 'A new comment.',
        ],
        [
            'message' => 'Another new comment.',
        ],
    ]);
    

<a name="updating-belongs-to-relationships"></a>

### Belongs To Relationships

Cuando actualizamos uni relation `belongsTo`, used suede utilizar el método `associate`. Este método establecerá la clave foránea en el model hijo.

    $account = App\Account::find(10);
    
    $user->account()->associate($account);
    
    $user->save();
    

When removing a `belongsTo` relationship, you may use the `dissociate` method. This method will set the relationship's foreign key to `null`:

    $user->account()->dissociate();
    
    $user->save();
    

<a name="updating-many-to-many-relationships"></a>

### Relaciones Muchos a Muchos

#### Adjuntar / Separar (Attaching / Detaching)

Eloquent also provides a few additional helper methods to make working with related models more convenient. Por ejemplo, imaginar que un usuario puede tener varios roles y un rol puede tener varios usuarios. Para adjuntar un rol a un usuario insertando un registro en la tabla intermedia que une los modelos, utilizar el método `attach`:

    $user = App\User::find(1);
    
    $user->roles()->attach($roleId);
    

Cuando se adjunta una relación a un modelo, se puede pasar además un array de datos adicional para insertarlo en la tabla intermedia:

    $user->roles()->attach($roleId, ['expires' => $expires]);
    

Por supuesto, a veces es necesario eliminar un rol de un usuario. Para eliminar un registro de una relación muchos-a-muchos, utilizar el método `detach`. El método `detach` eliminará el registro apropiado de la tabla intermedia; sin embargo, ambos modelos permanecerán en la base de datos:

    // Detach a single role from the user...
    $user->roles()->detach($roleId);
    
    // Detach all roles from the user...
    $user->roles()->detach();
    

Por comodidad, `attach` y `detach` aceptan además un array de IDs como entrada:

    $user = App\User::find(1);
    
    $user->roles()->detach([1, 2, 3]);
    
    $user->roles()->attach([
        1 => ['expires' => $expires],
        2 => ['expires' => $expires]
    ]);
    

#### Syncing Associations

Usted puede usar también el método `sync` para construir asociaciones muchos-a-muchos El método `sync` acepta una matriz de IDs para almacenar en la tabla intermedia (pivot) Cualquier ID que no esté contenido en la matriz será eliminado de la tabla intermedia So, after this operation is complete, only the IDs in the given array will exist in the intermediate table:

    $user->roles()->sync([1, 2, 3]);
    

Usted puede pasar además valores adicionales con los IDs para la tabla intermedia:

    $user->roles()->sync([1 => ['expires' => true], 2, 3]);
    

If you do not want to detach existing IDs, you may use the `syncWithoutDetaching` method:

    $user->roles()->syncWithoutDetaching([1, 2, 3]);
    

#### Toggling Associations

The many-to-many relationship also provides a `toggle` method which "toggles" the attachment status of the given IDs. If the given ID is currently attached, it will be detached. Likewise, if it is currently detached, it will be attached:

    $user->roles()->toggle([1, 2, 3]);
    

#### Saving Additional Data On A Pivot Table

Cuando trabajamos con una relación muchos-a-muchos, el método **Save** acepta , como segundo argumento, una matriz de attributes de la tabla anexa (tabla pivot)

    App\User::find(1)->roles()->save($role, ['expires' => $expires]);
    

#### Updating A Record On A Pivot Table

If you need to update an existing row in your pivot table, you may use `updateExistingPivot` method. This method accepts the pivot record foreign key and an array of attributes to update:

    $user = App\User::find(1);
    
    $user->roles()->updateExistingPivot($roleId, $attributes);
    

<a name="touching-parent-timestamps"></a>

## Touching Parent Timestamps

Cuando un modelo `belongsTo` (pertenece a) o `belongsToMany` (pertenece a muchos) otro modelo, tal como `Comment` pertenece a `Post`, puede ser útil actualizar los *timestamps* del padre cuando el hijo se actualiza. Por ejemplo, cuando un `Comment` se actualiza, se puede "tocar" (*touch*) el *timestamp* de `updated_at` del padre `Post`. Eloquent hace esto proceso muy sencillo. Simplemente añadir la propiedad `touches` conteniendo los nombres de las relaciones a tocar al modelo hijo:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Comment extends Model
    {
        /**
         * All of the relationships to be touched.
         *
         * @var array
         */
        protected $touches = ['post'];
    
        /**
         * Get the post that the comment belongs to.
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }
    

Ahora, al actualizar un `Comment`, el `Post` propietario actualizará también su columna `updated_at`, permitiendo saber cuando invalidar una caché del modelo `Post`:

    $comment = App\Comment::find(1);
    
    $comment->text = 'Edit to this comment!';
    
    $comment->save();