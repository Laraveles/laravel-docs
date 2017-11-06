# Eloquent: Mutators

- [Introducción](#introduction)
- [*Accessors* y *mutators*](#accessors-and-mutators) 
    - [Definiendo un *accessor*](#defining-an-accessor)
    - [Definiendo un *mutator*](#defining-a-mutator)
- [Mutators de fechas](#date-mutators)
- [Casting de atributos](#attribute-casting) 
    - [Casting Array & JSON](#array-and-json-casting)

<a name="introduction"></a>

## Introducción

Los *accessors* (accesores) y *mutators* (mutadores) permiten formatear los valores de los atributos de Eloquent cuando se accede a ellos o bien se establece o cambia su valor en una instancia. Por ejemplo, se puede usar [Laravel encrypter](/docs/{{version}}/encryption) para encriptar un valor mientras es almacenado en la base de datos, y luego automáticamente desencriptar el atributo cuando se acceder a él en un modelo de Eloquent.

Además de poder personalizar los *accessors* y *mutators*, Eloquent permite convertir campos de fechas a instancias de [Carbon](https://github.com/briannesbitt/Carbon) o incluso de [convertir campos texto a JSON](#attribute-casting).

<a name="accessors-and-mutators"></a>

## *Accessors* y *mutators*

<a name="defining-an-accessor"></a>

### Definiendo un *accessor*

Para definir un *accessor*, hay que crear un método `getFooAttribute` en el modelo donde `Foo` es el nombre "studly" de la columna a la que se desea acceder (importante las mayúsculas y minúsculas). En este ejemplo, se define un accessor para el atributo `first_name`. Eloquent llamará automáticamente al método *accessor* cuando se intente obtener el valor del atributo `first_name`:

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
    

Como se puede ver, el valor original de la columna es pasado al *accessor*, permite manipularlo y devolverlo. Para acceder al valor del *accessor*, únicamente hay que acceder al atributo `first_name` en una instancia del modelo:

    $user = App\User::find(1);
    
    $firstName = $user->first_name;
    

Por supuesto, se pueden utilizar *accesors* para devolver valores computados de atributos existentes:

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

Para definir un *mutator*, hay que crear un método `setFooAttribute` en el modelo donde `Foo` es el nombre "*studly*" de la columna a la que se desea acceder (importante mayúsculas y minúsculas). Así que ahora se va a definir un *mutator* para el atributo `first_name`. Eloquent llamará automáticamente a este *mutator* cuando se intente alterar el valor del atributo `first_name` del modelo:

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
    

El mutador recibirá el valor a modificar en el atributo, permitiendo la manipulación del mismo y guardarlo en la propiedad interna `$attributes`. Así por ejemplo, si se intenta guardar `Sally` en el atributo `first_name`:

    $user = App\User::find(1);
    
    $user->first_name = 'Sally';
    

En este ejemplo, la función `setFirstNameAttribute` será llamada con el valor `Sally`. El *mutador* intentará aplicar la función `strtolower` al nombre y guardará su valor en el *array* interno `$attributes`.

<a name="date-mutators"></a>

## *Mutators* de fechas

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
    

Cuando una columna se considera una fecha, se puede configurar su valor a un *timestamp de Unix*, una cadena de fecha (`Y-m-d`), una cadena *date-time* y por supuesto una instancia `DateTime` / `Carbon`. Las fechas serán automáticamente almacenadas correctamente en la base de datos:

    $user = App\User::find(1);
    
    $user->deleted_at = Carbon::now();
    
    $user->save();
    

Como se indicó anteriormente, cuando se recuperan atributos que son listados en la propiedad `dates`, automáticamente serán convertidos a instancias de [Carbon](https://github.com/briannesbitt/Carbon), permitiendo utilizar cualquier método de Carbon sobre los atributos:

    $user = App\User::find(1);
    
    return $user->deleted_at->getTimestamp();
    

#### Formatos de fecha

Por defecto, los *timestamps* tiene el formato `'Y-m-d H:i:s'`. Si se necesita personalizar el formato del *timestamp*, hay que configurar la propiedad `$dateFormat` del modelo. Esta propiedad determina como los atributos de fechas son almacenados en la base de datos, así como su formato cuando el modelo es serializado a un array o JSON:

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

La propiedad `$casts` del modelo proporciona un método adecuado para convertir atributos a tipos de datos comunes. La propiedad `$casts` debe contener un array donde la clave es el nombre del atributo a aplicar el *casting* y el valor el tipo de *casting* a realizar. Los tipos soportados para convertir son: `integer`, `real`, `float`, `double`, `string`, `boolean`, `object`, `array`, `collection`, `date`, `datetime`, y `timestamp`.

Por ejemplo, para convertir el atributo `is_admin`, el cual se almacena en la base de datos como un entero (`` o `1`) a un valor boobleano:

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
    

Ahora el atributo `is_admin` siempre será convertido a tipo *boolean* cuando se acceda a él, incluso si el valor subyacente es almacenado en la base de datos como un entero:

    $user = App\User::find(1);
    
    if ($user->is_admin) {
        //
    }
    

<a name="array-and-json-casting"></a>

### Casting Array & JSON

El tipo de conversión `array` es especialmente útil cuando se trabaja con columnas que están almacenadas como JSON serializados. Por ejemplo, si la base de datos tiene un campo de tipo `TEXT` o `JSON` que contiene un JSON serializado, añadiendo el *cast* `array` al atributo, automáticamente *deserializará* el atributo a un array de PHP cuando se acceda a él en un modelo Eloquent:

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
    

Una vez la conversión está definida, se puede tener acceso al atributo `options` y automáticamente será deserializado desde un JSON a un array de PHP. Cuando se establezca el valor del atributo `options`, el array dado será automáticamente serializado en JSON para su almacenamiento:

    $user = App\User::find(1);
    
    $options = $user->options;
    
    $options['key'] = 'value';
    
    $user->options = $options;
    
    $user->save();