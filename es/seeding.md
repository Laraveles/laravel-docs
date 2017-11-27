# Base de Datos: Poblado – *Seeding*

- [Introducción](#introduction)
- [Definir *seeders*](#writing-seeders) 
    - [Utilizar factorías – *factories*](#using-model-factories)
    - [Llamar a *seeders* adicionales](#calling-additional-seeders)
- [Ejecutar *seeders*](#running-seeders)

<a name="introduction"></a>

## Introducción

Laravel incluye un método simple para el poblado de la base de datos con datos de prueba utilizando clases "semilla" (*seeds*). Todas estas clases se almacenan en el directorio `database/seeds`. Las clases *seed* pueden tener cualquier nombre, pero normalmente deberían seguir una convención de nombres como `UsersTableSeeder`, etc. Por defecto, se incluye una clase `DatabaseSeeder`. Desde esta clase, se puede utilizar el método `call` para ejecutar otras clases de poblado, permitiendo controlar el orden.

<a name="writing-seeders"></a>

## Definir *seeders*

Para generar un *seeder*, ejecutar el [comando de Artisan](/docs/{{version}}/artisan) `make:seeder`. Todos los *seeders* que genera el framework se almacenarán en la carpeta `database/seeds`:

    php artisan make:seeder UsersTableSeeder
    

Una clase *seeder* únicamente contiene el método `run`. Se llamará a este método cuando se ejecute el [comando de Artisan](/docs/{{version}}/artisan) `db:seed`. En el método `run`, se puede insertar información en la base de datos de cualquier modo. Se puede utilizar el [query builder](/docs/{{version}}/queries) para insertar datos de forma manual o utilizar las [Eloquent model factories](/docs/{{version}}/database-testing#writing-factories) (factorías de modelos Eloquent).

Como ejemplo, se va a modificar la clase por defecto `DatabaseSeeder` para añadir una declaración *insert* al método `run`:

    <?php
    
    use Illuminate\Database\Seeder;
    use Illuminate\Database\Eloquent\Model;
    
    class DatabaseSeeder extends Seeder
    {
        /**
         * Run the database seeds.
         *
         * @return void
         */
        public function run()
        {
            DB::table('users')->insert([
                'name' => str_random(10),
                'email' => str_random(10).'@gmail.com',
                'password' => bcrypt('secret'),
            ]);
        }
    }
    

<a name="using-model-factories"></a>

### Utilizar factorías – *factories*

Por supuesto, especificar los atributos para cada modelo puede resultar tedioso. En su lugar, se pueden utilizar [factorías de modelos](/docs/{{version}}/database-testing#writing-factories) (*model factories*) para generar grandes cantidades de registros en la base de datos. Primero revise la documentación sobre [factorías de modelos](/docs/{{version}}/database-testing#writing-factories) para aprender a definir sus propias factorias. Una vez definidas, se puede utilizar la función *helper* `factory` para insertar registros en la base de datos.

Por ejemplo, para crear 50 usuarios y adjuntar una relación a cada usuario:

    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        factory(App\User::class, 50)->create()->each(function ($u) {
            $u->posts()->save(factory(App\Post::class)->make());
        });
    }
    

<a name="calling-additional-seeders"></a>

### Llamar a *seeders* adicionales

En la clase `DatabaseSeeder`, se puede utilizar el método `call` para ejecutar otras clases de poblado. Utilizar el método `call` permite dividir el poblado de la base de datos en varios archivos por lo que las clases de poblado no se vuelven abrumadoramente grandes. Simplemente pasar el nombre de la clase seeder a ejecutar:

    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $this->call([
            UsersTableSeeder::class,
            PostsTableSeeder::class,
            CommentsTableSeeder::class,
        ]);
    }
    

<a name="running-seeders"></a>

## Ejecutar *seeders*

Una vez definidas las clases *seeder*, se puede utilizar el comando de Artisan `db:seed` para poblar la base de datos. Por defecto, el comando `db:seed` ejecuta la clase `DatabaseSeeder`, que se puede utilizar para llamar a las otras clases *seed*. Sin embargo, se puede utilizar la opción `--class` para especificar una clase *seeder* concreta:

    php artisan db:seed
    
    php artisan db:seed --class=UsersTableSeeder
    

Se puede también poblar la base de datos utilizando el comando `migrate:refresh`, el cual hará un *rollback* (retroceso) y ejecutará todas las migraciones. Este comando resulta útil para reconstruir la base de datos completa:

    php artisan migrate:refresh --seed