# Base de Datos: Migraciones

- [Introduccion](#introduction)
- [Generar Migraciones](#generating-migrations)
- [Estructura de Una Migración](#migration-structure)
- [Ejecutar Migraciones](#running-migrations) 
    - [Revertir Migraciones](#rolling-back-migrations)
- [Tablas](#tables) 
    - [Crear Tablas](#creating-tables)
    - [Renombrando / Borrando Tablas](#renaming-and-dropping-tables)
- [Columnas](#columns) 
    - [Crear Columnas](#creating-columns)
    - [Modificadores de columnas](#column-modifiers)
    - [Modificar Columnas](#modifying-columns)
    - [Borrar Columnas](#dropping-columns)
- [Índices](#indexes) 
    - [Crear Índices](#creating-indexes)
    - [Borrar Índices](#dropping-indexes)
    - [Restricciones de Claves Ajenas](#foreign-key-constraints)

<a name="introduction"></a>

## Introducción

Migraciones son como un control de versión para tu base de datos, permitiéndole a tu equipo de manera sencilla modificar y compartir el esquema de la base de datos de tu aplicación. Las migraciones están normalmente emparejadas al constructor de esquemas de Laravel para construir fácilmente el esquema de la base de datos de una aplicación. Si alguna vez has tenido que decirle a un compañero de equipo que agregara manualmente una columna a su esquema de base de datos local, te has enfrentado a un problema que las migraciones de base de datos pueden resolver.

El `Schema` [facade](/docs/{{version}}/facades) de laravel proporciona soporte de base de datos independiente para crear y manipular tablas en todos los sistemas de bases de datos admitidos por Laravel.

<a name="generating-migrations"></a>

## Generar Migraciones

Para crear una migración, usa el [comando de Artisan](/docs/{{version}}/artisan) `make:migration`:

    php artisan make:migration create_users_table
    

Las nuevas migraciones serán almacenadas en el directorio `database/migrations`. Cada nombre de cada archivo de migración contiene un timestamp que permite a Laravel determinar el orden de las migraciones.

Las opciones `--table` and `--create` pueden utilizarse para indicar el nombre de la tabla y si la migración va a crear una nueva tabla. Estas opciones simplemente rellenan el archivo stub de la migración generado con la tabla especificada:

    php artisan make:migration create_users_table --create=users
    
    php artisan make:migration add_votes_to_users_table --table=users
    

Si se desea especificar una ruta de salida personalizada para la migración generada, se puede utilizar la opción `--path` al ejecutar el comando `make: migration`. La ruta de acceso proporcionada debe ser relativa a la ruta base de la aplicación.

<a name="migration-structure"></a>

## Estructura de una Migración

Una clase migración contiene dos métodos: `up` y `down`. El método `up` se utiliza para agregar nuevas tablas, columnas o índices a la base de datos, mientras que el método `down` debería simplemente invertir la operación realizada por el método `up`.

Dentro de estos métodos se puede utilizar el schema builder de Laravel para crear y modificar las tablas expresivamente. Para aprender más sobre los métodos disponibles en el `Schema`builder, [revisar la documentación](#creating-tables). Por ejemplo, esta migración crea una tabla `flights`:

    <?php
    
    use Illuminate\Support\Facades\Schema;
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Migrations\Migration;
    
    class CreateFlightsTable extends Migration
    {
        /**
         * Run the migrations.
         *
         * @return void
         */
        public function up()
        {
            Schema::create('flights', function (Blueprint $table) {
                $table->increments('id');
                $table->string('name');
                $table->string('airline');
                $table->timestamps();
            });
        }
    
        /**
         * Reverse the migrations.
         *
         * @return void
         */
        public function down()
        {
            Schema::drop('flights');
        }
    }
    

<a name="running-migrations"></a>

## Ejecutar Migraciones

Para correr todas las migraciones, ejecuta el comando de Artisan `migrate`:

    php artisan migrate
    

> {note} Si está usando la [Máquina Virtual Homestead](/docs/{{version}}/homestead), debe ejecutar este comando dentro de su maquina virtual.

#### Forzando Migraciones para Ejecutarlas en Producción

Algunas operaciones de migración son destructivas, esto quiere decir que puede causar pérdida de datos. Para protegerte de ejecutar estos comandos contra tu base de datos de producción, se te pedirá confirmación antes de ejecutar los comandos. Para forzar la ejecución de estos comandos sin confirmación, usa el parámetro `--force`:

    php artisan migrate --force
    

<a name="rolling-back-migrations"></a>

### Revertir Migraciones

Para revertir la última migración, debes usar el comando `rollback`. Este comando regresa al último "batch" de migraciones, que puede incluir múltiples archivos de migración:

    php artisan migrate:rollback
    

Puedes revertir un número limitado de migraciones colocando la opción `step` al comando `rollback`. Por ejemplo, el siguiente comando revertirá las últimas cinco migraciones:

    php artisan migrate:rollback --step=5
    

El comando `migrate:reset` deshará todas las migraciones de tu aplicación:

    php artisan migrate:reset
    

#### Revertir y Migrar en un Solo Comando

El comando `migrate:refresh` revertirá todas tus migraciones y luego ejecutará el comando `migrate`. Este comando re-creará efectivamente toda la base de datos:

    php artisan migrate:refresh
    
    // Refresh the database and run all database seeds...
    php artisan migrate:refresh --seed
    

Puede revertir & y volver a migrar un limitado número de migraciones proporcionando la opción `step` en el comando `refresh`. Por ejemplo, el siguiente comando revertirá & y volverá a migrar las últimas cinco migraciones:

    php artisan migrate:refresh --step=5
    

#### Eliminar Todas Las Tablas y Migrar

El comando `migrate:fresh` eliminará todas las tablas de la base de datos y luego ejecutará el comando `migrate`:

    php artisan migrate:fresh
    
    php artisan migrate:fresh --seed
    

<a name="tables"></a>

## Tablas

<a name="creating-tables"></a>

### Crear Tablas

Para crear una nueva tabla en la base de datos, usa el método `create` del facade `Schema`. El método `create` acepta dos argumentos. El primero es el nombre de la tabla, el segundo es un `Closuse` que recibe un objeto `Blueprint` usado para definir la nueva tabla:

    Schema::create('users', function (Blueprint $table) {
        $table->increments('id');
    });
    

Por supuesto, al crear una tabla, se puede hacer uso de los [métodos de columna](#creating-columns) del schema builder para definir las columnas de la tabla.

#### Comprobar la Existencia de Tablas / Columnas

Para comprobar la existencia de una tabla o columna se pueden utilizar los métodos `hasTable` y `hasColumn`:

    if (Schema::hasTable('users')) {
        //
    }
    
    if (Schema::hasColumn('users', 'email')) {
        //
    }
    

#### Conexión y Motor de almacenamiento

Si usted quiere realizar una operación en el esquema de una conexión a la base de datos que no es la conexión por defecto, utilice el método `connection`:

    Schema::connection('foo')->create('users', function (Blueprint $table) {
        $table->increments('id');
    });
    

Puedes usar la propiedad `engine` en el schema builder para definir el mecanismo de almacenamiento para la tabla:

    Schema::create('users', function (Blueprint $table) {
        $table->engine = 'InnoDB';
    
        $table->increments('id');
    });
    

<a name="renaming-and-dropping-tables"></a>

### Renombrando / Borrando Tablas

Para renombrar una tabla de la base de datos existente, puedes usar el método `rename`:

    Schema::rename($from, $to);
    

Para borrar una tabla existente, puedes usar los métodos `drop` o `dropIfExists`:

    Schema::drop('users');
    
    Schema::dropIfExists('users');
    

#### Renombrando Tablas con Claves Foráneas

Antes de renombrar una tabla, debes verificar que cualquier restricción de la llave foránea en la tabla tiene un nombre explícito en sus archivos de migración en lugar de dejar que Laravel asigne un nombre basado en la convención. De lo contrario, el nombre de la restricción de la llave foránea se referirá al nombre de la tabla anterior.

<a name="columns"></a>

## Columnas

<a name="creating-columns"></a>

### Crear Columnas

El metodo `table` del `Schema` facade puede ser usado para actualizar las tablas existentes. Al igual que el método `create`, el método `table` acepta dos argumentos: el nombre de la tabla y una `Closure` que recibe una instancia `Blueprint` que puede usar para agregar columnas a la tabla:

    Schema::table('users', function (Blueprint $table) {
        $table->string('email');
    });
    

#### Tipos de datos permitidos

Por supuesto, el Schema builder contiene una variedad de tipos de columna que puede especificar cuando este creando sus tablas:

| Comando                                       | Descripción                                                                                             |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `$table->bigIncrements('id');`             | Equivalente a un BIGINT SIN SIGNO (llave primaria) Auto-Incremento.                                     |
| `$table->bigInteger('votes');`             | Equivalente a un BIGINT.                                                                                |
| `$table->binary('data');`                  | Equivalente a un BLOB.                                                                                  |
| `$table->boolean('confirmed');`            | Equivalente a un BOOLEAN.                                                                               |
| `$table->char('name', 100);`               | Equivalente a un CHAR con longitud opcional.                                                            |
| `$table->date('created_at');`              | Equivalente a un DATE.                                                                                  |
| `$table->dateTime('created_at');`          | Equivalente a un DATETIME.                                                                              |
| `$table->dateTimeTz('created_at');`        | Equivalente a un DATETIME (con zona horaria).                                                           |
| `$table->decimal('amount', 8, 2);`         | Equivalente a un DECIMAL con precisión (dígitos totales) y escala (dígitos decimales).                  |
| `$table->double('amount', 8, 2);`          | Equivalente a un DOUBLE con precisión (dígitos totales) y escala (dígitos decimales).                   |
| `$table->enum('level', ['easy', 'hard']);` | Equivalente a un ENUM.                                                                                  |
| `$table->float('amount', 8, 2);`           | Equivalente a un FLOAT con precisión (dígitos totales) y escala (dígitos decimales).                    |
| `$table->geometry('positions');`           | Equivalente a un GEOMETRY.                                                                              |
| `$table->geometryCollection('positions');` | Equivalente a una GEOMETRYCOLLECTION.                                                                   |
| `$table->increments('id');`                | Equivalente a un INTEGER SIN SIGNO (llave primaria) Auto-Incremento.                                    |
| `$table->integer('votes');`                | Equivalente a un INTEGER.                                                                               |
| `$table->ipAddress('visitor');`            | Equivalente a una dirección IP.                                                                         |
| `$table->json('options');`                 | Equivalente a un JSON.                                                                                  |
| `$table->jsonb('options');`                | Equivalente a un JSONB.                                                                                 |
| `$table->lineString('positions');`         | Equivalente a un LINESTRING.                                                                            |
| `$table->longText('description');`         | Equivalente a un LONGTEXT.                                                                              |
| `$table->macAddress('device');`            | Equivalente a una dirección MAC.                                                                        |
| `$table->mediumIncrements('id');`          | Equivalente a un MEDIUMINT SIN SIGNO (llave primaria) Auto-Incremento.                                  |
| `$table->mediumInteger('votes');`          | Equivalente a un MEDIUMINT.                                                                             |
| `$table->mediumText('description');`       | Equivalente a un MEDIUMTEXT.                                                                            |
| `$table->morphs('taggable');`              | Equivalente a agregar un `taggable_id` INTEGER SIN SIGNO y `taggable_type` VARCHAR.                     |
| `$table->multiLineString('positions');`    | Equivalente a un MULTILINESTRING.                                                                       |
| `$table->multiPoint('positions');`         | Equivalente a un MULTIPOINT.                                                                            |
| `$table->multiPolygon('positions');`       | Equivalente a un MULTIPOLYGON.                                                                          |
| `$table->nullableMorphs('taggable');`      | Agrega versiones nulas de `morphs()`.                                                                   |
| `$table->nullableTimestamps();`            | Agrega versiones nulas de `timestamps()`.                                                               |
| `$table->point('position');`               | Equivalente a un POINT.                                                                                 |
| `$table->polygon('positions');`            | Equivalente a un POLYGON.                                                                               |
| `$table->rememberToken();`                 | Equivalente a agregar `remember_token` VARCHAR(100) que admite nulos.                                   |
| `$table->smallIncrements('id');`           | Equivalente a un SMALLINT SIN SIGNO (llave primaria) Auto-Incremento.                                   |
| `$table->smallInteger('votes');`           | Equivalente a un SMALLINT.                                                                              |
| `$table->softDeletes();`                   | Equivalente a un TIMESTAMP con campos null `deleted_at` para soft deletes.                              |
| `$table->softDeletesTz();`                 | Equivalente a un TIMESTAMP (con zona horaria) con campos null `deleted_at` para soft deletes.           |
| `$table->string('name', 100);`             | Equivalente a un VARCHAR con longitud opcional.                                                         |
| `$table->text('description');`             | Equivalente a un TEXT.                                                                                  |
| `$table->time('sunrise');`                 | Equivalente a un TIME.                                                                                  |
| `$table->timeTz('sunrise');`               | Equivalente a un TIME (con zona horaria).                                                               |
| `$table->timestamp('added_on');`           | Equivalente a un TIMESTAMP.                                                                             |
| `$table->timestampTz('added_on');`         | Equivalente a un TIMESTAMP (con zona horaria).                                                          |
| `$table->timestamps();`                    | Equivalente a agregar columnas nullable para `created_at` y `updated_at ` TIMESTAMP.                    |
| `$table->timestampsTz();`                  | Equivalente a agregar columnas nullable para `created_at` y `updated_at ` TIMESTAMP (con zona horaria). |
| `$table->tinyIncrements('id');`            | Equivalente a un TINYINT SIN SIGNO (llave primaria) Auto-Incremento.                                    |
| `$table->tinyInteger('votes');`            | Equivalente a un TINYINT.                                                                               |
| `$table->unsignedBigInteger('votes');`     | Equivalente a un BIGINT SIN SIGNO.                                                                      |
| `$table->unsignedDecimal('amount', 8, 2);` | Equivalente a un DECIMAL SIN SIGNO, con precisión (dígitos totales) y escala (dígitos decimales).       |
| `$table->unsignedInteger('votes');`        | Equivalente a un INTEGER SIN SIGNO.                                                                     |
| `$table->unsignedMediumInteger('votes');`  | Equivalente a un MEDIUMINT SIN SIGNO.                                                                   |
| `$table->unsignedSmallInteger('votes');`   | Equivalente a un SMALLINT SIN SIGNO.                                                                    |
| `$table->unsignedTinyInteger('votes');`    | Equivalente a un TINYINT SIN SIGNO.                                                                     |
| `$table->uuid('id');`                      | Equivalente a un UUID.                                                                                  |

<a name="column-modifiers"></a>

### Modificadores de columnas

Además de los tipos de columna enumerados anteriormente, hay varios "modificadores" de columna que puede usar al agregar una columna a una tabla de la base de datos. Por ejemplo, para hacer la columna "Nula" (nullable) puede utilizar el método `nullable`:

    Schema::table('users', function (Blueprint $table) {
        $table->string('email')->nullable();
    });
    

A continuación se muestra una lista de todos los modificadores de columna disponibles. Esta lista no incluye los [index modifiers](#creating-indexes):

| Modificador                         | Descripción                                                                        |
| ----------------------------------- | ---------------------------------------------------------------------------------- |
| `->after('column')`              | Coloca la columna después "after" de otra columna (solo MySQL)                     |
| `->autoIncrement()`              | Establece columnas INTEGER como auto-incremento (llave primaria)                   |
| `->charset('utf8')`              | Especifica un conjunto de caracteres para la columna (solo MySQL)                  |
| `->collation('utf8_unicode_ci')` | Especifica un cotejo para la columna (solo MySQL/SQL Server)                       |
| `->comment('my comment')`        | Agrega un comentario a una columna (solo MySQL)                                    |
| `->default($value)`              | Especifica un valor "por defecto" para la columna                                  |
| `->first()`                      | Coloca la columna como la primera en la tabla (sólo MySQL)                         |
| `->nullable($value = true)`      | Permite insertar valores nulos en la columna                                       |
| `->storedAs($expression)`        | Crear una columna generada virtual (solo MySQL)                                    |
| `->unsigned()`                   | Establece columnas INTEGER como SIN SIGNO (solo MySQL)                             |
| `->useCurrent()`                 | Establece columnas TIMESTAMP para usar CURRENT_TIMESTAMP como valor predeterminado |
| `->virtualAs($expression)`       | Crear una columna generada virtual (solo MySQL)                                    |

<a name="modifying-columns"></a>

### Modificar Columnas

#### Requisitos Previos

Antes de modificar una columna, asegúrate de añadir la dependencia `doctrine/dbal` en tu fichero `composer.json`. La librería "Doctrine DBAL" es usada para determinar el estado actual de la columna y crear las "consultas SQL" necesarias para hacer los ajustes específicos en la columna:

    composer require doctrine/dbal
    

#### Actualizando los atributos de las columnas

El método `change` permite modificar algunos tipos de columnas existentes a uno nuevo o cambiar los atributos de las columnas. Por ejemplo, puedes incrementar el tamaño de una columna de tipo string. Para ver funcionando el método `change`, vamos a incrementar el tamaño de la columna `name` desde 25 hasta 50:

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->change();
    });
    

También podríamos modificar una columna para que pueda contener nulos (nullable):

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->nullable()->change();
    });
    

> {note} Los siguientes tipos no se pueden "cambiar": char, double, enum, mediumInteger, timestamp, tinyInteger, ipAddress, json, jsonb, macAddress, mediumIncrements, morphs, nullableMorphs, nullableTimestamps, softDeletes, timeTz, timestampTz, timestamps, timestampsTz, unsignedMediumInteger, unsignedTinyInteger, uuid.

#### Renombrado columnas

Para renombrar una columna, puedes usar el método `renameColumn` en el constructor de esquemas (Schema Builder). Antes de renombrar la columna, asegúrate de añadir la dependencia `doctrine/dbal` en tu fichero `composer.json`:

    Schema::table('users', function (Blueprint $table) {
        $table->renameColumn('from', 'to');
    });
    

> {note} Renombrar cualquier columna en un tabla que tambien tenga una columna de tipo `enum` no esta soportado actualmente.

<a name="dropping-columns"></a>

### Borrar Columnas

Para borrar una columna, usa el método `dropColumn` en el Schema builder. Antes de borrar columnas desde una base de datos SQLite, debes agregar la dependencia `doctrine/dbal` en el archivo `composer.json` y ejecutar el comando `composer update` en la terminal para instalar la librería:

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn('votes');
    });
    

Puede borrar varias columnas de una tabla pasando una matriz con los nombres de las columnas usando el método `dropColumn`:

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn(['votes', 'avatar', 'location']);
    });
    

> {note} Eliminar o modificar múltiples columnas dentro de una sola migración mientras usa una base de datos SQLite no está soportado.

<a name="indexes"></a>

## Índices

<a name="creating-indexes"></a>

### Crear Índices

El constructor de esquemas (Schema Builder) soporta varios tipos de índices. Primero, veamos un ejemplo que especifica que los valores de una columnas deberían ser únicos. Para crear el índice podemos, simplemente, encadenar el método `unique` en la definición de la columna:

    $table->string('email')->unique();
    

Como alternativa, puedes definir el nombre de la columna después de definir el índice:

    $table->unique('email');
    

Incluso puedes pasar un array de columnas al método "index" para crear un índice compuesto:

    $table->index(['account_id', 'created_at']);
    

Laravel automáticamente generará un nombre razonable para el índice, pero puedes pasarle un segundo argumento al método para especificar el nombre:

    $table->unique('email', 'unique_email');
    

#### Tipos de índices disponibles

| Comando                                    | Descripción                             |
| ------------------------------------------ | --------------------------------------- |
| `$table->primary('id');`                | Agrega una llave primaria.              |
| `$table->primary(['id', 'parent_id']);` | Agrega llaves compuestas.               |
| `$table->unique('email');`              | Agrega un índice único.                 |
| `$table->index('state');`               | Agrega un índice simple.                |
| `$table->spatialIndex('location');`     | Agrega un índice espacial. (solo MySQL) |

#### Longitud de índices y MySQL / MariaDB

Laravel usa el conjunto de caracteres `utf8mb4` por defecto, que incluye soporte para almacenar "emojis" en la base de datos. Si estás ejecutando una versión de MySQL anterior a la versión 5.7.7 o MariaDB anterior a la versión 10.2.2, necesitas configurar manualmente la longitud de cadena predeterminada generada por las migraciones para que MySQL cree índices para ellas. Puede configurar esto llamando al método `Schema::defaultStringLength` dentro del `AppServiceProvider`:

    use Illuminate\Support\Facades\Schema;
    
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Schema::defaultStringLength(191);
    }
    

Alternativamente, puedes habilitar la opción `innodb_large_prefix` para tu base de datos. Consulta la documentación de tu base de datos para obtener instrucciones sobre cómo habilitar correctamente esta opción.

<a name="dropping-indexes"></a>

### Borrar Índices

Para borrar un índice, debe especificar el nombre del índice. Por defecto, Laravel asigna automáticamente un nombre lógico para los índices. Simplemente concatenando el nombre de la tabla, el nombre de la columna indexada y el tipo de el índice. Aquí tienes algunos ejemplos:

| Comando                                        | Descripción                                     |
| ---------------------------------------------- | ----------------------------------------------- |
| `$table->dropPrimary('users_id_primary');`  | Elimina una clave primaria de la tabla "users". |
| `$table->dropUnique('users_email_unique');` | Elimina un índice único de la tabla "users".    |
| `$table->dropIndex('geo_state_index');`     | Elimina un índice básico de la tabla "geo".     |

Si pasas un array de columnas dentro de un método que borra índices, el nombre convencional del índice se generará basado en el nombre de la tabla, las columnas y el tipo de llave:

    Schema::table('geo', function (Blueprint $table) {
        $table->dropIndex(['state']); // Drops index 'geo_state_index'
    });
    

<a name="foreign-key-constraints"></a>

### Restricciones de Llaves Foráneas

Laravel también provee soporte para la creación de restricciones de llaves foráneas, que son usadas para forzar la integridad referencial a nivel de la base de datos. Por ejemplo, definamos una columna `user_id` en la tabla `posts` que hace referencia a la columna `id` en la tabla `users`:

    Schema::table('posts', function (Blueprint $table) {
        $table->integer('user_id')->unsigned();
    
        $table->foreign('user_id')->references('id')->on('users');
    });
    

Puede especificar también la acción deseada para los atributos "on delete" (borrado) y "on update" (actualizado) de la restricción:

    $table->foreign('user_id')
          ->references('id')->on('users')
          ->onDelete('cascade');
    

Para eliminar una llave foránea, debe utilizar el método `dropForeign`. Las restricciones de llave foránea usan la misma convención que los índices. Así, concatenaremos el nombre de la tabla y las columnas en la restricción y le colocaremos el sufijo "_foreign":

    $table->dropForeign('posts_user_id_foreign');
    

O puedes pasar un array de valores que utilizará automáticamente el nombre convencional de la restricción cuando se esté borrando:

    $table->dropForeign(['user_id']);
    

Puede habilitar o deshabilitar restricciones de llave foránea en sus migraciones utilizando los siguientes métodos:

    Schema::enableForeignKeyConstraints();
    
    Schema::disableForeignKeyConstraints();