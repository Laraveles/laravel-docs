# Eloquent: Serialización

- [Introducción](#introduction)
- [*Serializar* modelos & colecciones](#serializing-models-and-collections) 
    - [*Serializar* a *arrays*](#serializing-to-arrays)
    - [*Serializar* a JSON](#serializing-to-json)
- [Ocultar atributos del JSON](#hiding-attributes-from-json)
- [Añadir valores al JSON](#appending-values-to-json)
- [*Serializar* fechas](#date-serialization)

<a name="introduction"></a>

## Introducción

Cuando se construyen APIs JSON, es común convertir modelos y sus relaciones en *arrays* o JSON. Eloquent incluye métodos para realizar estas conversiones, así como controlar qué atributos se incluyen en las serializaciones.

<a name="serializing-models-and-collections"></a>

## *Serializar* modelos & colecciones

<a name="serializing-to-arrays"></a>

### *Serializar* a *arrays*

Para convertir un modelo y sus [relationships](/docs/{{version}}/eloquent-relationships) en un *array*, debe usar el método `toArray`. Este método es recursivo, por lo que todos los atributos y relaciones (incluidas las relaciones de relaciones) se convertirán a *arrays*:

    $user = App\User::with('roles')->first();
    
    return $user->toArray();
    

Además también puede convertir modelos enteros de [colecciones](/docs/{{version}}/eloquent-collections) a *arrays*:

    $users = App\User::all();
    
    return $users->toArray();
    

<a name="serializing-to-json"></a>

### *Serializando* a JSON

Para convertir un modelo a JSON, debe simplemente utilizar el método `toJson`. Como `toArray`, el método `toJson` es recursivo, por lo que todos los atributos y relaciones se convertirán a JSON:

    $user = App\User::find(1);
    
    return $user->toJson();
    

De forma alternativa, puede convertir un modelo o colección a una cadena, la cual llamará automáticamente al método `toJson` sobre el modelo de la colección:

    $user = App\User::find(1);
    
    return (string) $user;
    

Puesto que los modelos y colecciones se convierten a JSON cuando se hace un casting de cadena, puede retornar objetos Eloquent directamente desde sus controladores o rutas de aplicación:

    Route::get('users', function () {
        return App\User::all();
    });
    

<a name="hiding-attributes-from-json"></a>

## Ocultar atributos del JSON

Algunas veces podría desear limitar los atributos, tales como contraseñas, que están incluídas en su representación JSON o *array* del modelo. Para hacerlo, agregue una propiedad `$hidden` a su modelo:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * The attributes that should be hidden for arrays.
         *
         * @var array
         */
        protected $hidden = ['password'];
    }
    

> {note} Al ocultar relaciones, use el nombre del método de la relación.

Por otro lado, se puede definir la propiedad `visible` para definir una lista blanca de atributos que deben ser incluidos en sus representaciones JSON y *array* del modelo. Todos los demás atributos estarán ocultos cuando el modelo se convierta en un *array* o JSON:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * The attributes that should be visible in arrays.
         *
         * @var array
         */
        protected $visible = ['first_name', 'last_name'];
    }
    

#### Modificar la visibilidad de atributos temporalmente

Si desea hacer visibles algunos atributos normalmente ocultos en una instancia de modelo determinada, puede usar el método `makeVisible`. El método `makeVisible` devuelve la instancia del modelo para un conveniente encadenamiento de métodos:

    return $user->makeVisible('attribute')->toArray();
    

Asimismo, si desea ocultar algunos atributos normalmente visibles en una instancia de modelo determinada, puede usar el método `makeHidden`.

    return $user->makeHidden('attribute')->toArray();
    

<a name="appending-values-to-json"></a>

## Añadir valores al JSON

Ocasionalmente, al convertir modelos a un *array* o JSON, puede querer agregar atributos que no tienen una columna correspodiente en su base de datos. Para ello, hay que definir un [accessor](/docs/{{version}}/eloquent-mutators) para el atributo:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * Get the administrator flag for the user.
         *
         * @return bool
         */
        public function getIsAdminAttribute()
        {
            return $this->attributes['admin'] == 'yes';
        }
    }
    

Después de crear el *accessor*, añada el nombre del atributo a la propiedad `appends` del modelo. Note que los nombres de los atributos se mencionan normalmente en "snake case", aunque el accessor se define usando "camel case":

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * The accessors to append to the model's array form.
         *
         * @var array
         */
        protected $appends = ['is_admin'];
    }
    

Una vez que el atributo se ha añadido a la lista `appends`, se incluirá tanto en las representaciones del modelo *array* como del modelo JSON. Los atributos en el *array* `appends` respetarán además la configuración de `visible` y `hidden` del modelo.

<a name="date-serialization"></a>

## Fecha de Serialización

Laravel extiende la biblioteca de fecha [Carbon](https://github.com/briannesbitt/Carbon) para proporcionar una personalización conveniente del formato de *serialización* JSON de Carbon. Para personalizar cómo se *serializan* todas las fechas de Carbon en su aplicación, use el método `Carbon::serializeUsing` method. El método `serializeUsing` acepta un *Closure* que devuelve una representación de la fecha en forma de cadena para la *serialización* JSON:

    <?php
    
    namespace App\Providers;
    
    use Illuminate\Support\Carbon;
    use Illuminate\Support\ServiceProvider;
    
    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Carbon::serializeUsing(function ($carbon) {
                return $carbon->format('U');
            });
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