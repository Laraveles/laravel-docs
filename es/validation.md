# Validación

- [Introducción](#introduction)
- [Comienzo rápido para la validación](#validation-quickstart) 
    - [Definir las rutas](#quick-defining-the-routes)
    - [Crear el controlador](#quick-creating-the-controller)
    - [Definir la lógica de validación](#quick-writing-the-validation-logic)
    - [Mostrar los errores de validación](#quick-displaying-the-validation-errors)
    - [Nota sobre los campos opcionales](#a-note-on-optional-fields)
- [Validación de la solicitud del formulario (*Form Request*)](#form-request-validation) 
    - [Creación de la solicitud del formulario](#creating-form-requests)
    - [Autorizacion de las solicitudes de formulario](#authorizing-form-requests)
    - [Personalizar los mensajes de error](#customizing-the-error-messages)
- [Crear validadores manualmente](#manually-creating-validators) 
    - [Redirección automática](#automatic-redirection)
    - [Nombrar a los contendores de errores](#named-error-bags)
    - [*Hook* después de la validación](#after-validation-hook)
- [Gestionar mensajes de error](#working-with-error-messages) 
    - [Mensajes de error personalizados](#custom-error-messages)
- [Reglas de validación disponibles](#available-validation-rules)
- [Agregar reglas condicionales](#conditionally-adding-rules)
- [Validación de Arrays](#validating-arrays)
- [Reglas de validación personalizadas](#custom-validation-rules) 
    - [Utilizar objetos de reglas](#using-rule-objects)
    - [Utilizar extensiones](#using-extensions)

<a name="introduction"></a>

## Introducción

Laravel incluye varias propuestas para validar la entrada de datos de su aplicación. Por defecto, la clase base del controlador de Laravel utiliza el *contrato (trait)* `ValidatesRequests` el cuál provee un método para validar la petición HTTP entrante con una gran variedad de reglas de validación muy potentes.

<a name="validation-quickstart"></a>

## Comienzo rápido con la validación

Para saber más sobre las características de las potentes reglas de validación, puede echar un vistazo a un ejemplo completo para validar un formulario y mostrar los mensajes de error al usuario.

<a name="quick-defining-the-routes"></a>

### Definir las rutas

Primero, asumiremos que tenemos definidas las siguientes rutas en el archivo `routes/web.php`:

    Route::get('post/create', 'PostController@create');
    
    Route::post('post', 'PostController@store');
    

Por supuesto, la ruta `GET` mostrará un formulario al usuario para crear un nuevo post en un blog, mientas que la ruta `POST` almacenará ese artículo en la base de datos.

<a name="quick-creating-the-controller"></a>

### Crear el controlador

A continuación, veamos un controlador simple que gestione estas rutas. Dejaremos el método `store` vacío por el momento:

    <?php
    
    namespace App\Http\Controllers;
    
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;
    
    class PostController extends Controller
    {
        /**
         * Show the form to create a new blog post.
         *
         * @return Response
         */
        public function create()
        {
            return view('post.create');
        }
    
        /**
         * Store a new blog post.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Validate and store the blog post...
        }
    }
    

<a name="quick-writing-the-validation-logic"></a>

### Escribiendo la lógica de validación

Ya estamos listos para incluir la lógica para validar el nuevo artículo en el método `store`. Para hacer esto, usaremos el método `validate` proporcionado por el objeto `Illuminate\Http\Request`. Si se pasa la regla de validación, el código continuará ejecutándose normalmente; sin embargo, si la regla falla, se lanzará una excepción y la respuesta apropiada será enviada, automáticamente, de vuelta al usuario. En el caso de una petición HTTP tradicional, se generará una respuesta de redirección, mientras que para peticiones AJAX se enviará una respuesta en formato JSON.

Para entender mejor el método `validate`, veamos el interior del método `store`:

    /**
     * Store a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $validatedData = $request->validate([
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);
    
        // The blog post is valid...
    }
    

Como se puede observar, simplemente se pasamos las reglas de validación deseadas al método `validate`. De nuevo, si la validación falla, será generada una apropiada respuesta de forma automática. Si la validación pasa, el controlador continuará ejecutándose con normalidad.

#### Detener la validación en el primer fallo

A veces puede que quiera deterner la validación en curso en un atributo después del primer error de validación. Para ello, asigne la regla `bail` al atributo:

    $request->validate([
        'title' => 'bail|required|unique:posts|max:255',
        'body' => 'required',
    ]);
    

En este ejemplo, si la regla `unique` en el atributo `title` falla, la regla de validación `max` no es comprobada. Las reglas son validadas en el orden que son asignadas.

#### Consideraciones sobre los atributos anidados

Si su petición HTTP contiene parámetros "anidados", puede especificarlos en las reglas de validación utilizando la notación de "puntos":

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'author.name' => 'required',
        'author.description' => 'required',
    ]);
    

<a name="quick-displaying-the-validation-errors"></a>

### Mostrar los errores de validación

Ahora, ¿qué sucede si los parámetros de validación entrantes no pasan las reglas de validación? Como se mencionó anteriormente, Laravel redireccionará automáticamente al usuario a su ubicación anterior. Además, todos los errores de validación serán *flashed* automáticamente a la [session](/docs/{{version}}/session#flash-data). 

> {note} Cuando se habla de flash, es en relación a un modo de guardar los errores en la sesión, y éstos serán eliminados cuando sean accedidos o esa sesión sea renovada.

De nuevo, observe que no teníamos que vincular explícitamente los mensajes de error a la vista en nuestra ruta `GET`. Esto se debe a que Laravel comprobará si hay errores en los datos de la sesión y automáticamente los enlazará a la vista si los datos están disponibles. La variable `$errors` será una instancia de `Illuminate\Support\MessageBag`. Para más información sobre cómo trabajar con este objeto, [eche un vistazo a su documentación](#working-with-error-messages).

> {tip} La variable `$errors` esta ligada a la vista a través del *middleware* `Illuminate\View\Middleware\ShareErrorsFromSession` el cual es proporcionado por el grupo de *middlewares* `web`. **Cuando este middleware es aplicado, la variable `$errors` estará siempre disponible en sus vistas**, permitiendo asumir convenientemente que la variable `$errors` siempre está definida y se puede ser usada con seguridad.

Así, en el ejemplo, el usuario será redirigido al método `create` de nuestro controlador cuando falle la validación, lo que nos permite mostrar los mensajes de error en la vista:

    <!-- /resources/views/post/create.blade.php -->
    
    <h1>Create Post</h1>
    
    @if ($errors->any())
        <div class="alert alert-danger">
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif
    
    <!-- Create Post Form -->
    

<a name="a-note-on-optional-fields"></a>

### Nota sobre los campos opcionales

Por defecto, Laravel incluye los *middleware* `TrimStrings` y `ConvertEmptyStringsToNull` en la pila global de *middlewares</en>. Estos *middlewares* son enumerados en la pila por la clase `App\Http\Kernel`. A causa de esto, a menudo, necesitará marcar la solicitud de campos "opcionales" como `nullable` si no quiere que el validador los considerelos valores `nulos (null)` como inválidos. 

A causa de esto, puede que necesitase colocar la regla <0>nullable</0> a los campos "opcionales" si no desea que el validador considere valores <0>null</0> como no válidos. Por ejemplo:</p> 

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);
    

En este ejemplo, estamos especificando que el campo `publish_at` puede ser `null` o la representación de una fecha válida. Si el modificador `nullable` no es añadido a la definición de la regla, el validador se consideraría `null` como una fecha invalida.

<a name="quick-ajax-requests-and-validation"></a>

#### Peticiones AJAX & Validación

En este ejemplo, utilizamos un formulario tradicional para enviar datos a la aplicación. Sin embargo, muchas aplicaciones utilizan peticiones AJAX. Cuando usamos el método `validate` durante una petición AJAX, Laravel no generará automáticamente una respuesta de redirección. En su lugar, genera una respuesta JSON conteniendo todos los errores de validación. Esta respuesta JSON se enviará con un código de estado HTTP 422.

<a name="form-request-validation"></a>

## Validación de peticiones (Form Request)

<a name="creating-form-requests"></a>

### Creación de peticiones de formularios

Para escenarios de validación mas complejos, se pueden crear "peticiones de formularios" ("form request"). Las peticiones de formularios son clases que contienen la lógica de la validación. Para crear una clase del tipo *form request*, utilice el comando Artisan `make:request` desde su consola:

    php artisan make:request StoreBlogPost
    

La clase generada será guardadad en el directorio `app/Http/Requests`. Si el directorio no existe, será creado automáticamente cuando ejecute el comando `make:request`. Agretemos unas pocas reglas de validación al método `rules`:

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ];
    }
    

Entonces, ¿cómo son evaluadas las reglas de validación? Todo lo que usted necesita hacer es especificar el tipo de la petición en su método controlador. La solicitud entrante del formulario es validada antes de que se llame al método del controlador, lo que significa que no necesita abarrotar su controlador con la lógica de validación:

    /**
     * Store the incoming blog post.
     *
     * @param  StoreBlogPost  $request
     * @return Response
     */
    public function store(StoreBlogPost $request)
    {
        // The incoming request is valid...
    }
    

Si la validación falla, se generará automáticamente una respuesta de redirección que envia al usuario de vuelta a su anterior localización. Los errores también se mostrarán "temporalmente" en la sesión así que están para su visualización. Si la petición era era una solicitud AJAX, se devolverá al usuario una respuesta HTTP con un código de estado 422 incluyendo una representación en formato JSON de los errores de validación.

#### Agregando *Hooks* después de las peticiones de formulario

Si se desea agregar un *hook* "posterior" a una petición de formulario (form request), puede usar el método `withValidator`. Este método recibe el validador totalmente construido, permitiéndole llamar a cualquiera de sus métodos antes que las reglas de validación sean realmente evaluadas:

    /**
     * Configure the validator instance.
     *
     * @param  \Illuminate\Validation\Validator  $validator
     * @return void
     */
    public function withValidator($validator)
    {
        $validator->after(function ($validator) {
            if ($this->somethingElseIsInvalid()) {
                $validator->errors()->add('field', 'Something is wrong with this field!');
            }
        });
    }
    

<a name="authorizing-form-requests"></a>

### Autorizacion de las peticiones de formulario

Las clases *form request* tambien contienen un metodo `authorize`. Dentro de este método, usted puede comprobar si el usuario autenticado tiene realmente el permiso para actualizar un recurso determinado. Por ejemplo, usted puede determinar si un usuario es, realmente, dueño de un comentario de un *blog* que esta intentando actualizar:

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        $comment = Comment::find($this->route('comment'));
    
        return $comment && $this->user()->can('update', $comment);
    }
    

Dado que todos los *Form Requests* extienden de la clase base *Request* de Laravel, se puede utilizar el método `user` para acceder al usuario autenticado. También observe la llamada al método `route` en el ejemplo anterior. Este método le garantiza el acceso a los parámetros de la URI definidos en la ruta que se está siendo llamada, como el parámetro `{comment}` en el siguiente ejemplo:

    Route::post('comment/{comment}');
    

Si el método `authorize` retorna `false`, es devuelta automáticamente una respuesta HTTP con un estado 403 y el método de su controlador no es ejecutado.

Si su plan es tener una lógica de autorización en otra parte de la aplicación, simplemente retorne `true` en el método `authorize`:

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }
    

<a name="customizing-the-error-messages"></a>

### Personalizar los mensajes de error

Usted puede personalizar los mensajes de error usados en la peitición de formulario sobreescribiendo el método `messages`. Este método debería retornar una matriz de pares de atributos / reglas y sus correspondientes mensajes de error:

    /**
     * Get the error messages for the defined validation rules.
     *
     * @return array
     */
    public function messages()
    {
        return [
            'title.required' => 'A title is required',
            'body.required'  => 'A message is required',
        ];
    }
    

<a name="manually-creating-validators"></a>

## Crear validadores manualmente

Si no desea utilizar el método `validate` dentro de una petición, usted podría crear manualmente, una instancia del validador utilizando el [facade](/docs/{{version}}/facades) `Validator`. El método `make` del *facade* genera una nueva instancia de validator:

    <?php
    
    namespace App\Http\Controllers;
    
    use Validator;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;
    
    class PostController extends Controller
    {
        /**
         * Store a new blog post.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $validator = Validator::make($request->all(), [
                'title' => 'required|unique:posts|max:255',
                'body' => 'required',
            ]);
    
            if ($validator->fails()) {
                return redirect('post/create')
                            ->withErrors($validator)
                            ->withInput();
            }
    
            // Store the blog post...
        }
    }
    

El primer argumento pasado al método `make` son los datos a validar. El segundo argumento es la regla de validación que debería ser aplicada a los datos.

Después de comprobar si la solicitud de validación falló, usted puede usar el método `withErrors` para *flash* los mensajes de error en la sesión. Cuando se usa ese método, la variable `$errors` será compartida automáticamente con sus vistas después de la redirección, permitiéndole mostrarlos fácilmente de nuevo al usuario. El método `withErrors` acepta un objeto *validator*, un `MessageBag` o una `matriz` de PHP.

<a name="automatic-redirection"></a>

### Redirección automática

Si desea crear una instancia del validador de manera manual y aun así aprovechar el re-direccionamiento automático ofrecido por el método `validate` de las peticiones, puede llamar el método `validate` en la instancia de tipo *validator*. De tal manera que si la validación falla, el usuario automáticamente sera redirigido, o en caso de que sea una petición AJAX, se retornara una respuesta en JSON:

    Validator::make($request->all(), [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ])->validate();
    

<a name="named-error-bags"></a>

### Nombrar a los *Error Bags*

Si se tiene múltiples formularios en una sola pagina, es posible querer darle nombre a los errores de `MessageBag`, lo que le permite recuperar los mensajes de error para un formulario específico. Esto se hace pasando un nombre como segundo argumento del método `withErrors`:

    return redirect('register')
                ->withErrors($validator, 'login');
    

Se puede acceder al nombre de la instancia de `MessageBag` desde la variable `$errors`:

    {{ $errors->login->first('email') }}
    

<a name="after-validation-hook"></a>

### *Hook* después de la validación

El validador también permite adjuntar *callbacks* para que se ejecuten después de que la validación se haya completado. Esto le permite realizar fácilmente una validación adicional e incluso agregar más mensajes de error a la colección de mensajes. Para usarlo, se utiliza el método `after` en una instancia del validador:

    $validator = Validator::make(...);
    
    $validator->after(function ($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });
    
    if ($validator->fails()) {
        //
    }
    

<a name="working-with-error-messages"></a>

## Gestionar mensajes de error

Después de llamar al método `errors` en una instancia `Validator`, se recibirá una instancia de `Illuminate\Support\MessageBag`, que tiene una variedad de métodos convenientes para trabajar con los mensajes de error. La variable `$errors` se encuentra disponible en todas las vista como una instancia de la clase `MessageBag`.

#### Recuperar el primer mensaje de error para un campo

Para recuperar el primer mensaje de error de un campo se utiliza el método `first`:

    $errors = $validator->errors();
    
    echo $errors->first('email');
    

#### Recuperar todos los mensajes de error para un campo

Si se desea recuperar un *array* de todos los mensajes para un campo determinado, se utiliza el método `get`:

    foreach ($errors->get('email') as $message) {
        //
    }
    

Si está validando un campo de formulario del tipo *array*, se pueden recuperar todos los mensajes para cada uno de los elementos del mismo utilizando el carácter `*`:

    foreach ($errors->get('attachments.*') as $message) {
        //
    }
    

#### Recuperar los mensajes de error para todos los campos

Para recuperar todos los mensajes de error se utiliza el método `all`:

    foreach ($errors->all() as $message) {
        //
    }
    

#### Verificar si existen mensajes para un campo

El método `has` determinara si existe algún mensaje de error para el campo determinado:

    if ($errors->has('email')) {
        //
    }
    

<a name="custom-error-messages"></a>

### Mensajes de error personalizados

Si es necesario, se pueden personalizar los mensajes de error de las validaciones en lugar de mostrar los que vienen por defecto. Hay varias maneras de especificar mensajes personalizados. La primera, es pasar los mensajes como tercer argumento del método `Validator::make`:

    $messages = [
        'required' => 'The :attribute field is required.',
    ];
    
    $validator = Validator::make($input, $rules, $messages);
    

En el ejemplo, se utiliza el *place-holder* `:attribute` que es reemplazado por el nombre del campo que se esta validando. Se pueden utilizar otros tipos de *place-holders* en los mensajes de validación. Por ejemplo:

    $messages = [
        'same'    => 'The :attribute and :other must match.',
        'size'    => 'The :attribute must be exactly :size.',
        'between' => 'The :attribute value :input is not between :min - :max.',
        'in'      => 'The :attribute must be one of the following types: :values',
    ];
    

#### Especificar un mensaje de personalizado para un atributo

Cuando se necesita, es posible personalizar un mensaje de error para un campo especifico. Se puede utilizar la notación "dot" o por "puntos". Se indica el nombre del atributo primero, seguido de la regla de validación:

    $messages = [
        'email.required' => 'We need to know your e-mail address!',
    ];
    

<a name="localization"></a>

#### Especificar un mensaje personalizado en los archivos de lenguaje

En la mayoría de los casos, probablemente quiera especificar el mensaje personalizado en el archivo de lenguaje en lugar de pasarlo directamente al `Validator`. Para hacerlo, se deben agregar los mensajes dentro del *array* `custom` del archivo de lenguaje ubicado en `resources/lang/xx/validation.php`.

    'custom' => [
        'email' => [
            'required' => 'We need to know your e-mail address!',
        ],
    ],
    

#### Especificar un atributo personalizado en los archivos de lenguaje

Si se desea cambiar la porción del mensaje de la validación correspondiente a `:attribute` para reemplazarlo con un nombre de atributo personalizado, se puede especificar el mismo en el *array* `attributes` del archivo de lenguaje `resources/lang/xx/validation.php`:

    'attributes' => [
        'email' => 'email address',
    ],
    

<a name="available-validation-rules"></a>

## Reglas de validación disponibles

A continuación se muestra una lista de todas las reglas de validación disponibles con su función:

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="collection-method-list">
  <p>
    <a href="#rule-accepted">Accepted</a> <a href="#rule-active-url">Active URL</a> <a href="#rule-after">After (Date)</a> <a href="#rule-after-or-equal">After Or Equal (Date)</a> <a href="#rule-alpha">Alpha</a> <a href="#rule-alpha-dash">Alpha Dash</a> <a href="#rule-alpha-num">Alpha Numeric</a> <a href="#rule-array">Array</a> <a href="#rule-before">Before (Date)</a> <a href="#rule-before-or-equal">Before Or Equal (Date)</a> <a href="#rule-between">Between</a> <a href="#rule-boolean">Boolean</a> <a href="#rule-confirmed">Confirmed</a> <a href="#rule-date">Date</a> <a href="#rule-date-equals">Date Equals</a> <a href="#rule-date-format">Date Format</a> <a href="#rule-different">Different</a> <a href="#rule-digits">Digits</a> <a href="#rule-digits-between">Digits Between</a> <a href="#rule-dimensions">Dimensions (Image Files)</a> <a href="#rule-distinct">Distinct</a> <a href="#rule-email">E-Mail</a> <a href="#rule-exists">Exists (Database)</a> <a href="#rule-file">File</a> <a href="#rule-filled">Filled</a> <a href="#rule-image">Image (File)</a> <a href="#rule-in">In</a> <a href="#rule-in-array">In Array</a> <a href="#rule-integer">Integer</a> <a href="#rule-ip">IP Address</a> <a href="#rule-json">JSON</a> <a href="#rule-max">Max</a> <a href="#rule-mimetypes">MIME Types</a> <a href="#rule-mimes">MIME Type By File Extension</a> <a href="#rule-min">Min</a> <a href="#rule-nullable">Nullable</a> <a href="#rule-not-in">Not In</a> <a href="#rule-numeric">Numeric</a> <a href="#rule-present">Present</a> <a href="#rule-regex">Regular Expression</a> <a href="#rule-required">Required</a> <a href="#rule-required-if">Required If</a> <a href="#rule-required-unless">Required Unless</a> <a href="#rule-required-with">Required With</a> <a href="#rule-required-with-all">Required With All</a> <a href="#rule-required-without">Required Without</a> <a href="#rule-required-without-all">Required Without All</a> <a href="#rule-same">Same</a> <a href="#rule-size">Size</a> <a href="#rule-string">String</a> <a href="#rule-timezone">Timezone</a> <a href="#rule-unique">Unique (Database)</a> <a href="#rule-url">URL</a>
  </p>
</div>

<a name="rule-accepted"></a>

#### accepted

El campo a validar debe contener *yes*, *no*, *1* o *true*. Puede ser útil para validar la aceptación de "Términos de servicio".

<a name="rule-active-url"></a>

#### active_url

El campo a validar debe contener un registro *A* o *AAAA* válido de acuerdo con la función de PHP `dns_get_record`.

<a name="rule-after"></a>

#### after:*date*

El campo a validar debe contener un valor posterior a una fecha concreta. Las fechas se pasarán a la función de PHP `strtotime`:

    'start_date' => 'required|date|after:tomorrow'
    

En lugar de pasar una fecha para ser evaluada por `strtotime`, se puede especificar otro campo con el que comparar la fecha:

    'finish_date' => 'required|date|after:start_date'
    

<a name="rule-after-or-equal"></a>

#### after*or\_equal:_date*

El campo bajo validación debe ser un valor posterior o igual a la fecha dada. Para obtener más información, consulte la regla [after](#rule-after).

<a name="rule-alpha"></a>

#### alpha

El campo a validar debe contener únicamente caracteres alfabéticos.

<a name="rule-alpha-dash"></a>

#### alpha_dash

El campo a validar debe contener caracteres alfa-numéricos, así como guiones altos y bajos.

<a name="rule-alpha-num"></a>

#### alpha_num

El campo a validar debe contener únicamente caracteres alfa-numéricos.

<a name="rule-array"></a>

#### array

El campo a validar debe ser un `array` PHP.

<a name="rule-before"></a>

#### before:*date*

El campo a validar debe contener una fecha anterior a la fecha dada. Las fechas serán pasadas a la función `strtotime` de PHP.

<a name="rule-before-or-equal"></a>

#### before*or\_equal:_date*

El campo a validar debe contener una fecha anterior a la fecha dada. Las fechas serán pasadas a la función `strtotime` de PHP.

<a name="rule-between"></a>

#### between:*min*,*max*

El campo a validar debe tener un tamaño entre *min* y *max*. Cadenas, caracteres numéricos, *arrays* y archivos se evalúan de la misma forma que la regla [`size`](#rule-size).

<a name="rule-boolean"></a>

#### boolean

El campo a validar debe poder ser transformado a un valor booleano. Los valores aceptados son `true`, `false`, `1`, ``, `"1"`, y `"0"`.

<a name="rule-confirmed"></a>

#### confirmed

El campo a validar debe coincidir con el campo `foo_confirmation`. Por ejemplo, si el campo fuera `password`, se debería proporcionar además un campo `password_confirmation`.

<a name="rule-date"></a>

#### date

El campo a validar debe contener una fecha válida de acuerdo con la función de PHP `strtotime`.

<a name="rule-date-equals"></a>

#### date_equals:*date*

El campo a validar debe contener una fecha igual a la fecha dada. Las fechas serán pasadas a la función `strtotime` de PHP.

<a name="rule-date-format"></a>

#### date_format:*format*

El campo a validar debe cumplir el *format* dado. Se puede usar **either** `date` o `date_format` cuando se valida un campo, no ambos.

<a name="rule-different"></a>

#### different:*field*

El campo a validar debe contener un valor diferente a *field*.

<a name="rule-digits"></a>

#### digits:*value*

El campo a validar debe ser *numérico* y una longitud exacta de *value*.

<a name="rule-digits-between"></a>

#### digits_between:*min*,*max*

El campo a validar debe tener un tamaño entre *min* y *max*.

<a name="rule-dimensions"></a>

#### dimensions

El archivo bajo validación debe ser una imagen que cumpla con las restricciones de dimensión especificadas dentro los parámetros de la regla:

    'avatar' => 'dimensions:min_width=100,min_height=200'
    

Las restricciones disponibles son: *min\_width*, *max\_width*, *min\_height*, *max\_height*, *width*, *height*, *ratio*.

El *ratio* debe representarse como el ancho dividido por la altura. Esto puede especificarse mediante una declaración como `3/2` o un flotante como `1.5 `:

    'avatar' => 'dimensions:ratio=3/2'
    

Dado que esta regla requiere varios argumentos, se puede utilizar el método `Rule::dimensions` para construir la regla con fluidez:

    use Illuminate\Validation\Rule;
    
    Validator::make($data, [
        'avatar' => [
            'required',
            Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
        ],
    ]);
    

<a name="rule-distinct"></a>

#### distinct

Cuando se trabaja con *arrays*, el campo a validar no debe tener valores duplicados.

    'foo.*.id' => 'distinct'
    

<a name="rule-email"></a>

#### email

El campo a validar debe contener un valor formateado como una dirección de correo electrónico.

<a name="rule-exists"></a>

#### exists:*table*,*column*

El campo a validar debe existir en una tabla de la base de datos.

#### Uso básico de la regla exists

    'state' => 'exists:states'
    

#### Especificar un nombre de columna

    'state' => 'exists:states,abbreviation'
    

Ocasionalmente, se puede necesitar especificar la conexión a base de datos a usar para la query de `exists`. Esto se puede lograr colocando antes del nombre de la tabla el nombre de la conexión, utilizando la sintaxis "dot" o por "punto":

    'email' => 'exists:connection.staff,email'
    

Si se desea personalizar la *consulta* que se ejecuta con la regla de validación, se puede usar la clase `Rule` para definir la regla. En el ejemplo, se especifican las reglas de validación como un *array* en lugar de utilizar el carácter `|` para delimitarlas:

    use Illuminate\Validation\Rule;
    
    Validator::make($data, [
        'email' => [
            'required',
            Rule::exists('staff')->where(function ($query) {
                $query->where('account_id', 1);
            }),
        ],
    ]);
    

<a name="rule-file"></a>

#### file

El campo a validar debe ser un archivo cargado con éxito.

<a name="rule-filled"></a>

#### filled

El campo a validar no debe estar vacío cuando se encuentra presente.

<a name="rule-image"></a>

#### image

El archivo a validar debe ser una imagen (jpeg, png, bmp, gif o svg)

<a name="rule-in"></a>

#### in:*foo*,*bar*,...

El campo a validar debe incluir alguno de los valores listados. Como esta regla a menudo requiere el `implode` de un *array*, `Rule::in` puede usarse para construir la regla con fluidez:

    use Illuminate\Validation\Rule;
    
    Validator::make($data, [
        'zones' => [
            'required',
            Rule::in(['first-zone', 'second-zone']),
        ],
    ]);
    

<a name="rule-in-array"></a>

#### in_array:*anotherfield*

En el campo a validar debe existir valores en *anotherfield*.

<a name="rule-integer"></a>

#### integer

El campo a validar debe ser un entero.

<a name="rule-ip"></a>

#### ip

El campo a validar debe contener una dirección IP.

#### ipv4

El campo a validar debe contener una dirección IPv4.

#### ipv6

El campo a validar debe contener una dirección IPv6.

<a name="rule-json"></a>

#### json

El campo a validar debe contener una cadena JSON válida.

<a name="rule-max"></a>

#### max:*value*

El campo a validar debe ser inferior o igual que un máximo *value*. Cadenas, caracteres numéricos, *arrays* y archivos se evalúan de la misma forma que la regla [`size`](#rule-size).

<a name="rule-mimetypes"></a>

#### mimetypes:*text/plain*,...

El campo a validar debe coincidir con uno de los tipos de MIME que se declaren:

    'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'
    

Para determinar el tipo de MIME del archivo cargado, se leerán los contenidos del archivo y el framework intentará adivinar el tipo MIME, que puede ser diferente del tipo MIME proporcionado por el cliente.

<a name="rule-mimes"></a>

#### mimes:*foo*,*bar*,...

El archivo a validar debe contener un tipo MIME que corresponda a una de las extensiones listadas.

#### Uso básico de la regla MIME

    'photo' => 'mimes:jpeg,bmp,png'
    

Aunque solo hay que especificar las extensiones, la regla realmente valida utilizando los tipos MIME del archivo leyendo el contenido del archivo y averiguando su tipo MIME.

La lista completa de tipos MIME y sus extensiones correspondientes se puede encontrar en el siguiente enlace: [http://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="rule-min"></a>

#### min:*value*

El campo a validar debe contener un valor mínimo de *value*. Cadenas de texto, números, *arrays* y archivos se evalúan del mismo modo que la regla [`size`](#rule-size).

<a name="rule-nullable"></a>

#### nullable

El campo a validar permite valores `null`. Esto es particularmente útil cuando se validan primitivas, como cadenas y enteros que pueden contener valores `null`.

<a name="rule-not-in"></a>

#### not_in:*foo*,*bar*,...

El campo a validar no debe estar incluido dentro del listado de valores. El método `Rule::notIn` se puede usar para escribir con fluidez la validación:

    use Illuminate\Validation\Rule;
    
    Validator::make($data, [
        'toppings' => [
            'required',
            Rule::notIn(['sprinkles', 'cherries']),
        ],
    ]);
    

<a name="rule-numeric"></a>

#### numeric

El campo a validar debe ser numérico.

<a name="rule-present"></a>

#### present

El campo bajo validación debe estar presente en los datos de entrada pero puede estar vacío.

<a name="rule-regex"></a>

#### regex:*pattern*

El campo a validar debe coincidir con la expresión regular pasada.

**Nota:** Cuando se utiliza el patrón `regex`, es posible que sea necesario especificar las reglas en un *array* en lugar de utilizar los delimitadores |, especialmente si la expresión regular contiene este carácter.

<a name="rule-required"></a>

#### required

El campo a validar debe estar presente en los datos proporcionados y nunca vacío. El campo se considera "vacío" si se cumple alguna de las siguientes condiciones:

<div class="content-list">
  <ul>
    <li>
      El valor es <code>null</code>.
    </li>
    <li>
      El valor es una cadena vacía.
    </li>
    <li>
      El valor es un <em>array</em> vacío o un objeto <code>Countable</code> vacío.
    </li>
    <li>
      El valor es un archivo subido sin directorio.
    </li>
  </ul>
</div>

<a name="rule-required-if"></a>

#### required_if:*anotherfield*,*value*,...

El campo a validar debe estar presente y no estar vació si el campo *anotherfield* es igual a algún *value*.

<a name="rule-required-unless"></a>

#### required_unless:*anotherfield*,*value*,...

El campo a validar debe estar presente y no estar vacío a no ser que el campo *anotherfield* sea igual a algún *value*.

<a name="rule-required-with"></a>

#### required_with:*foo*,*bar*,...

El campo a validar debe estar presente y no vacío *únicamente si* alguno de los otros campos están presentes.

<a name="rule-required-with-all"></a>

#### required_with_all:*foo*,*bar*,...

El campo a validar debe estar presente y no vacío *únicamente si* todos los otros campos están presentes.

<a name="rule-required-without"></a>

#### required_without:*foo*,*bar*,...

El campo a validar debe estar presente y no vacío *únicamente cuando* alguno de los otros campos no están presentes.

<a name="rule-required-without-all"></a>

#### required_without_all:*foo*,*bar*,...

El campo a validar debe estar presente y no vacío *únicamente cuando* todos campos no estén presentes.

<a name="rule-same"></a>

#### same:*field*

El campo *field* debe coincidir con el campo a validar.

<a name="rule-size"></a>

#### size:*value*

El campo a validar debe tener un tamaño *value*. Para cadenas, *value* corresponde al número de caracteres. Para datos numéricos, *value* corresponde con un valor entero. Para un *array*, *size* corresponde al `count` del *array*. Para archivos, *size* corresponde al tamaño en kilobytes.

<a name="rule-string"></a>

#### string

El campo de la validación debe ser una cadena de texto. Si se desea que el campo permita valores `null`, se debe asignar la regla `nullable` al campo.

<a name="rule-timezone"></a>

#### timezone

El campo a validar debe corresponderse con alguno de los identificadores de zona horaria según la función de PHP `timezone_identifiers_list`.

<a name="rule-unique"></a>

#### unique:*table*,*column*,*except*,*idColumn*

El campo a validar debe contener un valor único en una tabla de la base de datos. Si la opción `columna` no se especifica, se utilizará el nombre del campo.

**Especificar un nombre de columna:**

    'email' => 'unique:users,email_address'
    

**Seleccionar conexión de base de datos**

En ocasiones, puede ser necesario establecer la conexión a utilizar para las consultas realizadas por el Validator en la base de datos. Como en el ejemplo anterior, establecer `unique:users` como una regla de validación utilizará la conexión por defector para consultar la base de datos. Para reemplazar esto, especificar la conexión y el nombre de la tabla utilizando sintaxis de "puntos" o "dot notation":

    'email' => 'unique:connection.users,email_address'
    

**Forzar una regla *unique* para ignorar un ID:**

A veces es necesario ignorar un ID concreto en una comprobación *unique*. Por ejemplo, considerar una pantalla de "actualización de perfil" que incluye el nombre de usuario, e-mail y ubicación. Por supuesto, hay que verificar que la dirección de e-mail es única. Sin embargo, si el usuario solo cambia el campo nombre y no el campo e-mail, no será correcto si se lanza un error de validación por que el usuario es el propietario de esa dirección de e-mail.

Para indicar al validador que ignore el ID de usuario, se utilizará la clase `Rule` para definir la regla de forma fluida. En este ejemplo, se especificarán además las reglas de validación como *array* en lugar de utilizar el carácter `|` para delimitar las reglas:

    use Illuminate\Validation\Rule;
    
    Validator::make($data, [
        'email' => [
            'required',
            Rule::unique('users')->ignore($user->id),
        ],
    ]);
    

Si la tabla usa una clave primaria distinta a `id`, se puede especificar el nombre de la columna llamando al método `ignore`:

    'email' => Rule::unique('users')->ignore($user->id, 'user_id')
    

**Agregar cláusulas *Where* adicionales:**

Se pueden especificar además cláusulas adicionales personalizando la consulta utilizando el método `where`. Por ejemplo, añadir una cláusula que verifique que el `account_id` es `1`:

    'email' => Rule::unique('users')->where(function ($query) {
        return $query->where('account_id', 1);
    })
    

<a name="rule-url"></a>

#### url

El campo a validar debe ser una URL valida.

<a name="conditionally-adding-rules"></a>

## Agregar reglas condicionales

#### Validar únicamente si esta presente

En algunos casos, se pueden ejecutar validaciones a campos **únicamente** si ese campo está presente en los datos de entrada. Para ello, añadir la regla `sometimes` a la lista de reglas:

    $v = Validator::make($data, [
        'email' => 'sometimes|required|email',
    ]);
    

En el ejemplo anterior, el campo `email` será validado únicamente si está presente en el *array* `$data`.

> {tip} Si se intenta validar un campo que debe estar siempre presente pero puede estar vacío, revisar [esta nota para campos opcionales](#a-note-on-optional-fields).

#### Validación condicional compleja

A veces es necesario agregar reglas de validación basadas en lógica condicional más compleja. Por ejemplo, se puede requerir un campo si otro posee un valor mayor a 100. O, tal vez que un campo posea un valor concreto únicamente cuando otro campo está presente. Añadir estas reglas de validación debería ser sencillo. Primero, crear la instancia `Validator` con las *reglas estáticas* que no cambiarán:

    $v = Validator::make($data, [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);
    

Supongamos que nuestra aplicación es para coleccionistas de vídeo juegos. Si el coleccionista se registra en la aplicación y posee más de 100 juegos, requeriremos que explique por qué tiene tantos juegos. Por ejemplo, tal vez sea una tienda de reventa de juegos, o quizás simplemente disfruten de la colección. Para añadir condicionalmente este requerimiento, se puede utilizar el método `sometimes` de la instancia `Validator`.

    $v->sometimes('reason', 'required|max:500', function ($input) {
        return $input->games >= 100;
    });
    

El primer argumento pasado al método `sometimes` será el nombre del campo a validar de forma condicional. El segundo argumento serán las reglas de este campo. El tercer parámetro será un `Closure` que retornará `true` si las reglas deben tenerse en cuenta. Este método hace que crear validaciones condicionales complejas resulte muy sencillo. Se pueden añadir validaciones condicionales para varios campos a la vez:

    $v->sometimes(['reason', 'cost'], 'required', function ($input) {
        return $input->games >= 100;
    });
    

> {tip} El parámetro `$input` pasado al `Closure` será una instancia de `Illuminate\Support\Fluent` y se puede utilizar para acceder a los datos de entrada y archivos.

<a name="validating-arrays"></a>

## Validación de Arrays

Validar un *array* basado en campos de un formulario no debería ser complicado. Se puede usar "dot notation" para validar atributos que sean *array*. Por ejemplo, si la petición HTTP contiene en campo `photos[profile]`, se puede validar de la siguiente forma:

    $validator = Validator::make($request->all(), [
        'photos.profile' => 'required|image',
    ]);
    

También se puede validar cada elemento de un *array*. Por ejemplo, para validar que cada correo electrónico en un campo determinado del *array* es único, se puede hacer lo siguiente:

    $validator = Validator::make($request->all(), [
        'person.*.email' => 'email|unique:users',
        'person.*.first_name' => 'required_with:person.*.last_name',
    ]);
    

Del mismo modo, se puede usar el carácter `*` para especificar los mensajes de validación en los archivos de idioma, por lo que es muy fácil utilizar un solo mensaje de validación para campos basados en *array*:

    'custom' => [
        'person.*.email' => [
            'unique' => 'Each person must have a unique e-mail address',
        ]
    ],
    

<a name="custom-validation-rules"></a>

## Reglas de validación personalizadas

<a name="using-rule-objects"></a>

### Usando objetos *Rule*

Laravel incluye varias reglas de validación muy útiles; sin embargo, se pueden especificar reglas propias. Una de los métodos para registrar las reglas de validación personalizadas es usando los *rule objects*. Para generar un nuevo *rule object*, se puede usar el comando `make:rule` de Artisan. Se usara este comando para generar una regla que verifique que una cadena esté en mayúscula. Laravel ubicara la nueva regla en el directorio `app/Rules`:

    php artisan make:rule Uppercase
    

Una vez que el objeto ha sido creado, se puede definir su comportamiento. Un *rule object* contiene dos métodos: `passes` y `message`. El método `passes` recibe el valor del atributo y el nombre, y retorna `true` o `false` dependiendo si el valor del atributo es válido o no. El método `message` retorna el mensaje de error de la validación que debe ser usado cuando la misma falla:

    <?php
    
    namespace App\Rules;
    
    use Illuminate\Contracts\Validation\Rule;
    
    class Uppercase implements Rule
    {
        /**
         * Determine if the validation rule passes.
         *
         * @param  string  $attribute
         * @param  mixed  $value
         * @return bool
         */
        public function passes($attribute, $value)
        {
            return strtoupper($value) === $value;
        }
    
        /**
         * Get the validation error message.
         *
         * @return string
         */
        public function message()
        {
            return 'The :attribute must be uppercase.';
        }
    }
    

Por supuesto, se puede usar el helper `trans` desde su método `message` si desea devolver un mensaje de error desde los archivos de traducción:

    /**
     * Get the validation error message.
     *
     * @return string
     */
    public function message()
    {
        return trans('validation.uppercase');
    }
    

Una vez que se ha definido una regla, se puede asociar al validador pasando una instancia del *rule object* junto a las otras reglas:

    use App\Rules\Uppercase;
    
    $request->validate([
        'name' => ['required', new Uppercase],
    ]);
    

<a name="using-extensions"></a>

### Utilizar extensiones

Otra de las maneras de registrar reglas de validación personalizadas es utilizando el método `extend` del [facade](/docs/{{version}}/facades) `Validator`. Utilizaremos este método en un [service provider](/docs/{{version}}/providers) para registrar una regla de validación personalizada:

    <?php
    
    namespace App\Providers;
    
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Support\Facades\Validator;
    
    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Validator::extend('foo', function ($attribute, $value, $parameters, $validator) {
                return $value == 'foo';
            });
        }
    
        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }
    

El *Closure* del validador personalizado recibe cuatro argumentos: el nombre del `$attribute` a validar, el `$value` del atributo, un *array* de los `$parameters` que son pasados a la regla, y una instancia de `Validator`.

También se puede pasar una clase y método al método `extend` en lugar de un *Closure*:

    Validator::extend('foo', 'FooValidator@validate');
    

#### Definir el mensaje de error

Hay que definir un mensaje de error para la regla personalizada. Se puede hacer o pasando un mensaje concreto en el *array* o añadiendo una nueva entrada en el archivo de validación de lenguaje. Este mensaje debe incluirse en el primer nivel del *array*, nunca dentro del *array* `custom`, el cual es únicamente para mensajes de error de atributos específicos:

    "foo" => "Your input was invalid!",
    
    "accepted" => "The :attribute must be accepted.",
    
    // The rest of the validation error messages...
    

Cuando se crea una nueva regla de validación, a veces es necesario definir reemplazos personalizados para los mensajes de error. Esto se puede hacer añadiendo un Validator personalizado como se describe a continuación y llamando al método `replacer` de la *facade* `Validator`. Esto se podría incluir en el método `boot` de un [service provider](/docs/{{version}}/providers):

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Validator::extend(...);
    
        Validator::replacer('foo', function ($message, $attribute, $rule, $parameters) {
            return str_replace(...);
        });
    }
    

#### Extensiones implícitas

Por defecto, cuando se intenta validar un atributo que no está presente o contiene un valor vacío como se define en la regla [`required`](#rule-required), las reglas de validacion normales, incluidas las extensiones, no se ejecutan. Por ejemplo, la regla [`unique`](#rule-unique) no se ejecutara con un valor `null`:

    $rules = ['name' => 'unique'];
    
    $input = ['name' => null];
    
    Validator::make($input, $rules)->passes(); // true
    

Para que una regla se ejecute incluso cuando un atributo está vacío, la regla debe presuponer que el atributo es requerido. Para crear una extensión "implícita", utilizar el método `Validator::extendImplicit()`:

    Validator::extendImplicit('foo', function ($attribute, $value, $parameters, $validator) {
        return $value == 'foo';
    });
    

> {note} Note: Una extensión "implícita" únicamente *implica* que el atributo es requerido. Si se valida o no un atributo vacío queda a merced del usuario.