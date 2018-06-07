# Pruebas de Base de Datos

- [Introducción](#introduction)
- [Generating Factories](#generating-factories)
- [Restablecer la base de datos después de cada test](#resetting-the-database-after-each-test)
- [Writing Factories](#writing-factories) 
    - [Factory States](#factory-states)
- [Using Factories](#using-factories) 
    - [Creación de modelos](#creating-models)
    - [Persistencia de modelos](#persisting-models)
    - [Relaciones](#relationships)
- [Verificaciones disponibles](#available-assertions)

<a name="introduction"></a>

## Introducción

Laravel proporciona una variedad de herramientas útiles para hacer más fácil el *testing* de aplicaciones basadas en bases de datos. En primer lugar se puede usar el *helper* `assertDatabaseHas` para confirmar que los datos existentes en la base de datos coinciden con un conjunto determinado de criterios. Por ejemplo, si se desea verificar que hay un registro en la tabla `users` con el valor `email` de `sally@example.com`, se puede hacer lo siguiente:

    public function testDatabase()
    {
        // Make call to application...
    
        $this->assertDatabaseHas('users', [
            'email' => 'sally@example.com'
        ]);
    }
    

También se puede usar el *helper* `assertDatabaseMissing` para verificar que los datos no existen en la base de datos.

Desde luego, el método `assertDatabaseHas` y otros *helpers* como el se usan a conveniencia. Se es libre de usar cualquiera de los métodos de verificación integrados de PHPUnit para complementar las pruebas.

<a name="generating-factories"></a>

## Generating Factories

Para crear una *factory*, se usa el [comando de Artisan](/docs/{{version}}/artisan) `make:factory`:

    php artisan make:factory PostFactory
    

La nueva *factory* será puesta in su directorio `database/factories`.

La opción `--model` puede ser usada para indicar el nombre del modelo creado por la *factory*. Esta opción se pre-llenará al archivo de *factory* generado con el modelo dado:

    php artisan make:factory PostFactory --model=Post
    

<a name="resetting-the-database-after-each-test"></a>

## Restablecimiento de la Base de Datos tras Testing

A menudo es útil restablecer su base de datos después de cada prueba para que los datos de una prueba anterior no interfieran con las pruebas posteriores. El trait `RefreshDatabase` toma el enfoque más óptimo para migrar su base de datos de prueba dependiendo de si está usando una base de datos in-memory o una base de datos tradicional. Simplemente use el trait en su clase de prueba y todo se manejará por usted:

    <?php
    
    namespace Tests\Feature;
    
    use Tests\TestCase;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    
    class ExampleTest extends TestCase
    {
        use RefreshDatabase;
    
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->get('/');
    
            // ...
        }
    }
    

<a name="writing-factories"></a>

## Escritura de Fábricas

Al realizar la prueba, podría necesitar insertar unos pocos registros en su base de datos antes de ejecutar su prueba. En lugar de especificar manualmente el valor de cada columna cuando crea estos datos de prueba, Laravel le permite definir un set de configuración de atributos para cada uno de sus [Eloquent models](/docs/{{version}}/eloquent) usando *factories* modelo. Para empezar, chequée el archivo `database/factories/UserFactory.php` en su aplicación. Listo para su uso, este archivo contiene una definición de *factory*:

    use Faker\Generator as Faker;
    
    $factory->define(App\User::class, function (Faker $faker) {
        static $password;
    
        return [
            'name' => $faker->name,
            'email' => $faker->unique()->safeEmail,
            'password' => $password ?: $password = bcrypt('secret'),
            'remember_token' => str_random(10),
        ];
    });
    

Dentro del Closure, que sirve como definición de *factory*, puede devolver los valores de prueba predeterminados de todos los atributos en el modelo. El Closure recibirá una instancia de la librería PHP [Faker](https://github.com/fzaninotto/Faker), la cual le permite generar a conveniencia varios tipos de datos al azar para probar.

También puede crear archivos de *factory* adicionales por cada modelo para una mejor organización. Por ejemplo, podría crear archivos `UserFactory.php` y `CommentFactory.php` dentro de su directorio `database/factories`. Todos los archivos dentro del directorio `factories` se cargarán automáticamente por Laravel.

<a name="factory-states"></a>

### Estados de Fábrica

Los estados le permiten definir modificaciones discretas que pueden ser aplicadas a sus *factories* modelo en cualquier combinación. Por ejemplo, su modelo `User` puede tener un estado `delinquent` que modifica uno de sus valores de atributo predeterminado. Puede definir sus transformaciones de estado usando el método `state`. Para estados simples, puede pasar un array de modificaciones de atributos:

    $factory->state(App\User::class, 'delinquent', [
        'account_status' => 'delinquent',
    ]);
    

Si su estado requiere cálculo o una instancia `$faker`, puede usar un Closure para calcular las modificaciones de atributo del estado:

    $factory->state(App\User::class, 'address', function ($faker) {
        return [
            'address' => $faker->address,
        ];
    });
    

<a name="using-factories"></a>

## Uso de Fábricas

<a name="creating-models"></a>

### Creación de Modelos

Una vez que ha definido sus *factories*, puede usar la función global `factory` en sus pruebas o archivos poblados para generar instancias modelo. Entonces, echemos un vistazo a algunos ejemplos de creación de modelos. Primero, usaremos el método `make` para crear modelos pero sin guardarlos en la base de datos:

    public function testDatabase()
    {
        $user = factory(App\User::class)->make();
    
        // Use model in tests...
    }
    

También puede crear una Colección de algunos modelos o crear modelos de un tipo dado:

    // Create three App\User instances...
    $users = factory(App\User::class, 3)->make();
    

#### Aplicación de Estados

Puede también aplicar cualquiera de sus [states](#factory-states) a sus modelos. Si desea aplicar múltiples trasformaciones de estado a los modelos, deberá especificar el nombre de cada estado que desee aplicar:

    $users = factory(App\User::class, 5)->states('delinquent')->make();
    
    $users = factory(App\User::class, 5)->states('premium', 'delinquent')->make();
    

#### Reemplazo de Atributos

Si desea reemplazar alguno de los valores por defecto de sus modelos, puede pasar un array de valores al método `make`. Solo los valores especificados serán reemplazados mientras que el resto de los valores permanecen establecidos en sus valores predeterminados según lo especificado por la *factory*:

    $user = factory(App\User::class)->make([
        'name' => 'Abigail',
    ]);
    

<a name="persisting-models"></a>

### Persistencia de Modelos

El método `create` no solo crea las instancias modelo sino que también los guarda en su base de datos usando el método de Eloquent `save`:

    public function testDatabase()
    {
        // Create a single App\User instance...
        $user = factory(App\User::class)->create();
    
        // Create three App\User instances...
        $users = factory(App\User::class, 3)->create();
    
        // Use model in tests...
    }
    

Puede reemplazar atrributos sobre el modelo pasando un array al método `create`:

    $user = factory(App\User::class)->create([
        'name' => 'Abigail',
    ]);
    

<a name="relationships"></a>

### Relaciones

En este ejemplo, adjuntaremos una relación a unos modelos creados. Al usar el método `create` para crear múltiples modeos, una [collection instance](/docs/{{version}}/eloquent-collections) de Eloquent is devuelta, permitiéndole usar cualquiera de las funciones convenientes proporcionadas por la colección, tal y como `each`:

    $users = factory(App\User::class, 3)
               ->create()
               ->each(function ($u) {
                    $u->posts()->save(factory(App\Post::class)->make());
                });
    

#### Relaciones & y Atributos de Closures

Puede también enfocar las relaciones a modelos usando atributos Closure en sus definiciones de *factory*. Por ejemplo, si desea crear una nueva instancia de `User` al crear un `Post`, puede hacer lo siguiente:

    $factory->define(App\Post::class, function ($faker) {
        return [
            'title' => $faker->title,
            'content' => $faker->paragraph,
            'user_id' => function () {
                return factory(App\User::class)->create()->id;
            }
        ];
    });
    

Estos Closures también reciben el array de atributo evaluado de la *factory* que los define:

    $factory->define(App\Post::class, function ($faker) {
        return [
            'title' => $faker->title,
            'content' => $faker->paragraph,
            'user_id' => function () {
                return factory(App\User::class)->create()->id;
            },
            'user_type' => function (array $post) {
                return App\User::find($post['user_id'])->type;
            }
        ];
    });
    

<a name="available-assertions"></a>

## Verificaciones Disponibles

Laravel proporciona varias afirmaciones de base de datos de sus pruebas [PHPUnit](https://phpunit.de/):

| Método                                                  | Descripción                                                           |
| ------------------------------------------------------- | --------------------------------------------------------------------- |
| `$this->assertDatabaseHas($table, array $data);`     | Asegúrate que una tabla en la base de datos contiene los datos dados. |
| `$this->assertDatabaseMissing($table, array $data);` | Asegúrate que una tabla en la base de datos no contiene datos dados.  |
| `$this->assertSoftDeleted($table, array $data);`     | Asegúrate de que el registro dado haya sido borrado en forma suave.   |