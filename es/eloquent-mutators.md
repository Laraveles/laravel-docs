# Eloquent: Mutators

- [Introducción](#introduction)
- [Accessors y Mutators](#accessors-and-mutators) 
    - [Definiendo un Accessor](#defining-an-accessor)
    - [Definiendo un Mutator](#defining-a-mutator)
- [Mutators de Fechas](#date-mutators)
- [Casting de Atributos](#attribute-casting) 
    - [Casting Array & JSON](#array-and-json-casting)

<a name="introduction"></a>

## Introducción

Los *accessors* (accesores) y *mutators* (mutadores) permiten formatear los valores de los atributos de Eloquent cuando se accede a ellos o bien se establece o cambia su valor en una instancia. Por ejemplo, se puede usar [Laravel encrypter](/docs/{{version}}/encryption) para encriptar un valor mientras es almacenado en la base de datos, y luego automáticamente desencriptar el atributo cuando se acceder a él en un modelo de Eloquent.

Además de poder personalizar los accessors y mutators, Eloquent permite convertir campos de fechas a instancias de [Carbon](https://github.com/briannesbitt/Carbon) o incluso de [convertir campos texto a JSON](#attribute-casting).

<a name="accessors-and-mutators"></a>

## Accessors y Mutators

<a name="defining-an-accessor"></a>

### Definiendo un Accessor

Para definir un accessor, hay que crear un método `getFooAttribute` en el modelo donde `Foo` es el nombre "studly" de la columna a la que se desea acceder (importante las mayúsculas y minúsculas). En este ejemplo, se define un accessor para el atributo `first_name`. Eloquent llamará automáticamente al método accessor cuando se intente obtener el valor del atributo `first_name`:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * Get the user's first name.
         *
         * @param  string  $value
         * @return string
         */
        public function getFirstNameAttribute($value)
        {
            return ucfirst($value);
        }
    }
    

Como se puede ver, el valor original de la columna es pasado al accessor, permite manipularlo y devolverlo. Para acceder al valor del accessor, únicamente hay que acceder al atributo `first_name` en una instancia del modelo:

    $user = App\User::find(1);
    
    $firstName = $user->first_name;
    

Of course, you may also use accessors to return new, computed values from existing attributes:

    /**
     * Get the user's full name.
     *
     * @return string
     */
    public function getFullNameAttribute()
    {
        return "{$this->first_name} {$this->last_name}";
    }
    

<a name="defining-a-mutator"></a>

### Definiendo un Mutator

To define a mutator, define a `setFooAttribute` method on your model where `Foo` is the "studly" cased name of the column you wish to access. So, again, let's define a mutator for the `first_name` attribute. This mutator will be automatically called when we attempt to set the value of the `first_name` attribute on the model:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * Set the user's first name.
         *
         * @param  string  $value
         * @return void
         */
        public function setFirstNameAttribute($value)
        {
            $this->attributes['first_name'] = strtolower($value);
        }
    }
    

The mutator will receive the value that is being set on the attribute, allowing you to manipulate the value and set the manipulated value on the Eloquent model's internal `$attributes` property. So, for example, if we attempt to set the `first_name` attribute to `Sally`:

    $user = App\User::find(1);
    
    $user->first_name = 'Sally';
    

In this example, the `setFirstNameAttribute` function will be called with the value `Sally`. The mutator will then apply the `strtolower` function to the name and set its resulting value in the internal `$attributes` array.

<a name="date-mutators"></a>

## Mutadores de Fechas

Por defecto, Eloquent convertirá las columnas `created_at` y `updated_at` en instancias de [Carbon](https://github.com/briannesbitt/Carbon), las cuales proporcionan una gran variedad de métodos útiles y heredan de la clase nativa de PHP `DateTime`. Se puede personalizar qué campos de fecha deben ser automáticamente mutados, e incluso completamente desactivar esta opción sobrescribiendo la propiedad `$dates` del modelo:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * The attributes that should be mutated to dates.
         *
         * @var array
         */
        protected $dates = [
            'created_at',
            'updated_at',
            'deleted_at'
        ];
    }
    

When a column is considered a date, you may set its value to a UNIX timestamp, date string (`Y-m-d`), date-time string, and of course a `DateTime` / `Carbon` instance, and the date's value will automatically be correctly stored in your database:

    $user = App\User::find(1);
    
    $user->deleted_at = Carbon::now();
    
    $user->save();
    

As noted above, when retrieving attributes that are listed in your `$dates` property, they will automatically be cast to [Carbon](https://github.com/briannesbitt/Carbon) instances, allowing you to use any of Carbon's methods on your attributes:

    $user = App\User::find(1);
    
    return $user->deleted_at->getTimestamp();
    

#### Formatos de Fecha

By default, timestamps are formatted as `'Y-m-d H:i:s'`. If you need to customize the timestamp format, set the `$dateFormat` property on your model. This property determines how date attributes are stored in the database, as well as their format when the model is serialized to an array or JSON:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Flight extends Model
    {
        /**
         * The storage format of the model's date columns.
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }
    

<a name="attribute-casting"></a>

## Casting de Atributos

The `$casts` property on your model provides a convenient method of converting attributes to common data types. The `$casts` property should be an array where the key is the name of the attribute being cast and the value is the type you wish to cast the column to. The supported cast types are: `integer`, `real`, `float`, `double`, `string`, `boolean`, `object`, `array`, `collection`, `date`, `datetime`, and `timestamp`.

For example, let's cast the `is_admin` attribute, which is stored in our database as an integer (`` or `1`) to a boolean value:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * The attributes that should be cast to native types.
         *
         * @var array
         */
        protected $casts = [
            'is_admin' => 'boolean',
        ];
    }
    

Now the `is_admin` attribute will always be cast to a boolean when you access it, even if the underlying value is stored in the database as an integer:

    $user = App\User::find(1);
    
    if ($user->is_admin) {
        //
    }
    

<a name="array-and-json-casting"></a>

### Casting Array & JSON

The `array` cast type is particularly useful when working with columns that are stored as serialized JSON. For example, if your database has a `JSON` or `TEXT` field type that contains serialized JSON, adding the `array` cast to that attribute will automatically deserialize the attribute to a PHP array when you access it on your Eloquent model:

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * The attributes that should be cast to native types.
         *
         * @var array
         */
        protected $casts = [
            'options' => 'array',
        ];
    }
    

Once the cast is defined, you may access the `options` attribute and it will automatically be deserialized from JSON into a PHP array. When you set the value of the `options` attribute, the given array will automatically be serialized back into JSON for storage:

    $user = App\User::find(1);
    
    $options = $user->options;
    
    $options['key'] = 'value';
    
    $user->options = $options;
    
    $user->save();