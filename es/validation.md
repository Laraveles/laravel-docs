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
- [Validación de matrices](#validating-arrays)
- [Reglas de validación personalizadas](#custom-validation-rules) 
    - [Utilizar objetos de reglas](#using-rule-objects)
    - [Utilizar extensiones](#using-extensions)

<a name="introduction"></a>

## Introducción

Laravel provides several different approaches to validate your application's incoming data. Por defecto, la clase base del controlador de Laravel utiliza el *contrato (trait)* `ValidatesRequests` el cuál provee un método para validar la petición HTTP entrante con una gran variedad de reglas de validación muy potentes.

<a name="validation-quickstart"></a>

## Comienzo rápido con la validación

To learn about Laravel's powerful validation features, let's look at a complete example of validating a form and displaying the error messages back to the user.

<a name="quick-defining-the-routes"></a>

### Defining The Routes

First, let's assume we have the following routes defined in our `routes/web.php` file:

    Route::get('post/create', 'PostController@create');
    
    Route::post('post', 'PostController@store');
    

Of course, the `GET` route will display a form for the user to create a new blog post, while the `POST` route will store the new blog post in the database.

<a name="quick-creating-the-controller"></a>

### Creating The Controller

Next, let's take a look at a simple controller that handles these routes. We'll leave the `store` method empty for now:

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

Now we are ready to fill in our `store` method with the logic to validate the new blog post. Para hacer esto, usaremos el método `validate` proporcionado por el objeto `Illuminate\Http\Request`. If the validation rules pass, your code will keep executing normally; however, if validation fails, an exception will be thrown and the proper error response will automatically be sent back to the user. En el caso de una petición HTTP tradicional, se generará una respuesta de redirección, mientras que para peticiones AJAX se enviará una respuesta en formato JSON.

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
    

Como se puede observar, simplemente se pasamos las reglas de validación deseadas al método `validate`. De nuevo, si la validación falla, será generada una apropiada respuesta de forma automática. If the validation passes, our controller will continue executing normally.

#### Stopping On First Validation Failure

A veces puede que quiera deterner la validación en curso en un atributo después del primer error de validación. Para ello, asigne la regla `bail` al atributo:

    $request->validate([
        'title' => 'bail|required|unique:posts|max:255',
        'body' => 'required',
    ]);
    

In this example, if the `unique` rule on the `title` attribute fails, the `max` rule will not be checked. Rules will be validated in the order they are assigned.

#### Consideraciones sobre los atributos anidados

If your HTTP request contains "nested" parameters, you may specify them in your validation rules using "dot" syntax:

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'author.name' => 'required',
        'author.description' => 'required',
    ]);
    

<a name="quick-displaying-the-validation-errors"></a>

### Displaying The Validation Errors

Ahora, ¿qué sucede si los parámetros de validación entrantes no pasan las reglas de validación? Como se mencionó anteriormente, Laravel redireccionará automáticamente al usuario a su ubicación anterior. In addition, all of the validation errors will automatically be [flashed to the session](/docs/{{version}}/session#flash-data).

De nuevo, observe que no teníamos que vincular explícitamente los mensajes de error a la vista en nuestra ruta `GET`. Esto se debe a que Laravel comprobará si hay errores en los datos de la sesión y automáticamente lso enlazará a la vista si los datos están disponibles. La variable `$errors` será una instancia de `Illuminate\Support\MessageBag`. Para más información sobre cómo trabajar con este objeto, [eche un vistazo a su documentación](#working-with-error-messages).

> {tip} La variable `$errors` esta ligada a la vista a través del *middleware* `Illuminate\View\Middleware\ShareErrorsFromSession` el cual es proporcionado por el grupo de *middlewares* `web`. **Cuando este middleware es aplicado, la variable `$errors` estará siempre disponible en sus vistas**, permitiendo asumir convenientemente que la variable `$errors` siempre está definida y se puede ser usada con seguridad.

So, in our example, the user will be redirected to our controller's `create` method when validation fails, allowing us to display the error messages in the view:

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

### A Note On Optional Fields

By default, Laravel includes the `TrimStrings` and `ConvertEmptyStringsToNull` middleware in your application's global middleware stack. These middleware are listed in the stack by the `App\Http\Kernel` class. Because of this, you will often need to mark your "optional" request fields as `nullable` if you do not want the validator to consider `null` values as invalid. Por ejemplo:

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);
    

In this example, we are specifying that the `publish_at` field may be either `null` or a valid date representation. If the `nullable` modifier is not added to the rule definition, the validator would consider `null` an invalid date.

<a name="quick-ajax-requests-and-validation"></a>

#### AJAX Requests & Validation

In this example, we used a traditional form to send data to the application. However, many applications use AJAX requests. When using the `validate` method during an AJAX request, Laravel will not generate a redirect response. Instead, Laravel generates a JSON response containing all of the validation errors. This JSON response will be sent with a 422 HTTP status code.

<a name="form-request-validation"></a>

## Form Request Validation

<a name="creating-form-requests"></a>

### Creating Form Requests

Para escenarios mas complejos, se pueden crear "form request". Los Form Request son clases para las peticiones que contienen la lógica de la validación. To create a form request class, use the `make:request` Artisan CLI command:

    php artisan make:request StoreBlogPost
    

The generated class will be placed in the `app/Http/Requests` directory. If this directory does not exist, it will be created when you run the `make:request` command. Let's add a few validation rules to the `rules` method:

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
    

Entonces, ¿cómo se evalúan las reglas de validación? Todo lo que se necesita es colocar el request dentro del metodo del controlador. The incoming form request is validated before the controller method is called, meaning you do not need to clutter your controller with any validation logic:

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
    

If validation fails, a redirect response will be generated to send the user back to their previous location. Los errores también se mostrarán en la sesión para que estén disponibles para su visualización. Si el request era una solicitud AJAX, se devolverá una respuesta HTTP con un código de estado 422 al usuario incluyendo una representación en formato JSON de los errores de validación.

#### Adding After Hooks To Form Requests

If you would like to add an "after" hook to a form request, you may use the `withValidator` method. This method receives the fully constructed validator, allowing you to call any of its methods before the validation rules are actually evaluated:

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

### Authorizing Form Requests

The form request class also contains an `authorize` method. Within this method, you may check if the authenticated user actually has the authority to update a given resource. For example, you may determine if a user actually owns a blog comment they are attempting to update:

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
    

Since all form requests extend the base Laravel request class, we may use the `user` method to access the currently authenticated user. Also note the call to the `route` method in the example above. This method grants you access to the URI parameters defined on the route being called, such as the `{comment}` parameter in the example below:

    Route::post('comment/{comment}');
    

If the `authorize` method returns `false`, a HTTP response with a 403 status code will automatically be returned and your controller method will not execute.

Si el plan es tener una lógica de autorización en otra parte de la aplicación, simplemente se coloca return `true` en el método `authorize`:

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

### Customizing The Error Messages

You may customize the error messages used by the form request by overriding the `messages` method. This method should return an array of attribute / rule pairs and their corresponding error messages:

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

## Manually Creating Validators

If you do not want to use the `validate` method on the request, you may create a validator instance manually using the `Validator` [facade](/docs/{{version}}/facades). The `make` method on the facade generates a new validator instance:

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
    

El primer argumento pasado al método `make` es la data a validar, el segundo argumento es la regla de validación que debe ser aplicada a esta data.

After checking if the request validation failed, you may use the `withErrors` method to flash the error messages to the session. Cuando se usa ese método, la variable `$erros` es automáticamente compartida con las vistas después de la redirección, permitiendo mostrarlos fácilmente de nuevo al usuario. The `withErrors` method accepts a validator, a `MessageBag`, or a PHP `array`.

<a name="automatic-redirection"></a>

### Automatic Redirection

If you would like to create a validator instance manually but still take advantage of the automatic redirection offered by the requests's `validate` method, you may call the `validate` method on an existing validator instance. If validation fails, the user will automatically be redirected or, in the case of an AJAX request, a JSON response will be returned:

    Validator::make($request->all(), [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ])->validate();
    

<a name="named-error-bags"></a>

### Named Error Bags

If you have multiple forms on a single page, you may wish to name the `MessageBag` of errors, allowing you to retrieve the error messages for a specific form. Simply pass a name as the second argument to `withErrors`:

    return redirect('register')
                ->withErrors($validator, 'login');
    

You may then access the named `MessageBag` instance from the `$errors` variable:

    {{ $errors->login->first('email') }}
    

<a name="after-validation-hook"></a>

### After Validation Hook

The validator also allows you to attach callbacks to be run after validation is completed. This allows you to easily perform further validation and even add more error messages to the message collection. To get started, use the `after` method on a validator instance:

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

## Working With Error Messages

After calling the `errors` method on a `Validator` instance, you will receive an `Illuminate\Support\MessageBag` instance, which has a variety of convenient methods for working with error messages. The `$errors` variable that is automatically made available to all views is also an instance of the `MessageBag` class.

#### Retrieving The First Error Message For A Field

To retrieve the first error message for a given field, use the `first` method:

    $errors = $validator->errors();
    
    echo $errors->first('email');
    

#### Retrieving All Error Messages For A Field

If you need to retrieve an array of all the messages for a given field, use the `get` method:

    foreach ($errors->get('email') as $message) {
        //
    }
    

If you are validating an array form field, you may retrieve all of the messages for each of the array elements using the `*` character:

    foreach ($errors->get('attachments.*') as $message) {
        //
    }
    

#### Retrieving All Error Messages For All Fields

To retrieve an array of all messages for all fields, use the `all` method:

    foreach ($errors->all() as $message) {
        //
    }
    

#### Determining If Messages Exist For A Field

The `has` method may be used to determine if any error messages exist for a given field:

    if ($errors->has('email')) {
        //
    }
    

<a name="custom-error-messages"></a>

### Custom Error Messages

If needed, you may use custom error messages for validation instead of the defaults. There are several ways to specify custom messages. First, you may pass the custom messages as the third argument to the `Validator::make` method:

    $messages = [
        'required' => 'The :attribute field is required.',
    ];
    
    $validator = Validator::make($input, $rules, $messages);
    

In this example, the `:attribute` place-holder will be replaced by the actual name of the field under validation. You may also utilize other place-holders in validation messages. Por ejemplo:

    $messages = [
        'same'    => 'The :attribute and :other must match.',
        'size'    => 'The :attribute must be exactly :size.',
        'between' => 'The :attribute value :input is not between :min - :max.',
        'in'      => 'The :attribute must be one of the following types: :values',
    ];
    

#### Specifying A Custom Message For A Given Attribute

Sometimes you may wish to specify a custom error messages only for a specific field. You may do so using "dot" notation. Specify the attribute's name first, followed by the rule:

    $messages = [
        'email.required' => 'We need to know your e-mail address!',
    ];
    

<a name="localization"></a>

#### Specifying Custom Messages In Language Files

In most cases, you will probably specify your custom messages in a language file instead of passing them directly to the `Validator`. To do so, add your messages to `custom` array in the `resources/lang/xx/validation.php` language file.

    'custom' => [
        'email' => [
            'required' => 'We need to know your e-mail address!',
        ],
    ],
    

#### Specifying Custom Attributes In Language Files

If you would like the `:attribute` portion of your validation message to be replaced with a custom attribute name, you may specify the custom name in the `attributes` array of your `resources/lang/xx/validation.php` language file:

    'attributes' => [
        'email' => 'email address',
    ],
    

<a name="available-validation-rules"></a>

## Available Validation Rules

Below is a list of all available validation rules and their function:

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

The field under validation must be *yes*, *on*, *1*, or *true*. This is useful for validating "Terms of Service" acceptance.

<a name="rule-active-url"></a>

#### active_url

The field under validation must have a valid A or AAAA record according to the `dns_get_record` PHP function.

<a name="rule-after"></a>

#### after:*date*

The field under validation must be a value after a given date. The dates will be passed into the `strtotime` PHP function:

    'start_date' => 'required|date|after:tomorrow'
    

Instead of passing a date string to be evaluated by `strtotime`, you may specify another field to compare against the date:

    'finish_date' => 'required|date|after:start_date'
    

<a name="rule-after-or-equal"></a>

#### after*or\_equal:_date*

The field under validation must be a value after or equal to the given date. For more information, see the [after](#rule-after) rule.

<a name="rule-alpha"></a>

#### alpha

The field under validation must be entirely alphabetic characters.

<a name="rule-alpha-dash"></a>

#### alpha_dash

The field under validation may have alpha-numeric characters, as well as dashes and underscores.

<a name="rule-alpha-num"></a>

#### alpha_num

The field under validation must be entirely alpha-numeric characters.

<a name="rule-array"></a>

#### array

The field under validation must be a PHP `array`.

<a name="rule-before"></a>

#### before:*date*

The field under validation must be a value preceding the given date. The dates will be passed into the PHP `strtotime` function.

<a name="rule-before-or-equal"></a>

#### before*or\_equal:_date*

The field under validation must be a value preceding or equal to the given date. The dates will be passed into the PHP `strtotime` function.

<a name="rule-between"></a>

#### between:*min*,*max*

The field under validation must have a size between the given *min* and *max*. Strings, numerics, arrays, and files are evaluated in the same fashion as the [`size`](#rule-size) rule.

<a name="rule-boolean"></a>

#### boolean

The field under validation must be able to be cast as a boolean. Accepted input are `true`, `false`, `1`, ``, `"1"`, and `"0"`.

<a name="rule-confirmed"></a>

#### confirmed

The field under validation must have a matching field of `foo_confirmation`. For example, if the field under validation is `password`, a matching `password_confirmation` field must be present in the input.

<a name="rule-date"></a>

#### date

The field under validation must be a valid date according to the `strtotime` PHP function.

<a name="rule-date-equals"></a>

#### date_equals:*date*

The field under validation must be equal to the given date. The dates will be passed into the PHP `strtotime` function.

<a name="rule-date-format"></a>

#### date_format:*format*

The field under validation must match the given *format*. You should use **either** `date` or `date_format` when validating a field, not both.

<a name="rule-different"></a>

#### different:*field*

The field under validation must have a different value than *field*.

<a name="rule-digits"></a>

#### digits:*value*

The field under validation must be *numeric* and must have an exact length of *value*.

<a name="rule-digits-between"></a>

#### digits_between:*min*,*max*

The field under validation must have a length between the given *min* and *max*.

<a name="rule-dimensions"></a>

#### dimensions

The file under validation must be an image meeting the dimension constraints as specified by the rule's parameters:

    'avatar' => 'dimensions:min_width=100,min_height=200'
    

Available constraints are: *min\_width*, *max\_width*, *min\_height*, *max\_height*, *width*, *height*, *ratio*.

A *ratio* constraint should be represented as width divided by height. This can be specified either by a statement like `3/2` or a float like `1.5`:

    'avatar' => 'dimensions:ratio=3/2'
    

Since this rule requires several arguments, you may use the `Rule::dimensions` method to fluently construct the rule:

    use Illuminate\Validation\Rule;
    
    Validator::make($data, [
        'avatar' => [
            'required',
            Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
        ],
    ]);
    

<a name="rule-distinct"></a>

#### distinct

When working with arrays, the field under validation must not have any duplicate values.

    'foo.*.id' => 'distinct'
    

<a name="rule-email"></a>

#### email

The field under validation must be formatted as an e-mail address.

<a name="rule-exists"></a>

#### exists:*table*,*column*

The field under validation must exist on a given database table.

#### Basic Usage Of Exists Rule

    'state' => 'exists:states'
    

#### Specifying A Custom Column Name

    'state' => 'exists:states,abbreviation'
    

Occasionally, you may need to specify a specific database connection to be used for the `exists` query. You can accomplish this by prepending the connection name to the table name using "dot" syntax:

    'email' => 'exists:connection.staff,email'
    

If you would like to customize the query executed by the validation rule, you may use the `Rule` class to fluently define the rule. In this example, we'll also specify the validation rules as an array instead of using the `|` character to delimit them:

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

The field under validation must be a successfully uploaded file.

<a name="rule-filled"></a>

#### filled

The field under validation must not be empty when it is present.

<a name="rule-image"></a>

#### image

The file under validation must be an image (jpeg, png, bmp, gif, or svg)

<a name="rule-in"></a>

#### in:*foo*,*bar*,...

The field under validation must be included in the given list of values. Since this rule often requires you to `implode` an array, the `Rule::in` method may be used to fluently construct the rule:

    use Illuminate\Validation\Rule;
    
    Validator::make($data, [
        'zones' => [
            'required',
            Rule::in(['first-zone', 'second-zone']),
        ],
    ]);
    

<a name="rule-in-array"></a>

#### in_array:*anotherfield*

The field under validation must exist in *anotherfield*'s values.

<a name="rule-integer"></a>

#### integer

The field under validation must be an integer.

<a name="rule-ip"></a>

#### ip

The field under validation must be an IP address.

#### ipv4

The field under validation must be an IPv4 address.

#### ipv6

The field under validation must be an IPv6 address.

<a name="rule-json"></a>

#### json

The field under validation must be a valid JSON string.

<a name="rule-max"></a>

#### max:*value*

The field under validation must be less than or equal to a maximum *value*. Strings, numerics, arrays, and files are evaluated in the same fashion as the [`size`](#rule-size) rule.

<a name="rule-mimetypes"></a>

#### mimetypes:*text/plain*,...

The file under validation must match one of the given MIME types:

    'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'
    

To determine the MIME type of the uploaded file, the file's contents will be read and the framework will attempt to guess the MIME type, which may be different from the client provided MIME type.

<a name="rule-mimes"></a>

#### mimes:*foo*,*bar*,...

The file under validation must have a MIME type corresponding to one of the listed extensions.

#### Basic Usage Of MIME Rule

    'photo' => 'mimes:jpeg,bmp,png'
    

Even though you only need to specify the extensions, this rule actually validates against the MIME type of the file by reading the file's contents and guessing its MIME type.

A full listing of MIME types and their corresponding extensions may be found at the following location: <https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types>

<a name="rule-min"></a>

#### min:*value*

The field under validation must have a minimum *value*. Strings, numerics, arrays, and files are evaluated in the same fashion as the [`size`](#rule-size) rule.

<a name="rule-nullable"></a>

#### nullable

The field under validation may be `null`. This is particularly useful when validating primitive such as strings and integers that can contain `null` values.

<a name="rule-not-in"></a>

#### not_in:*foo*,*bar*,...

The field under validation must not be included in the given list of values. The `Rule::notIn` method may be used to fluently construct the rule:

    use Illuminate\Validation\Rule;
    
    Validator::make($data, [
        'toppings' => [
            'required',
            Rule::notIn(['sprinkles', 'cherries']),
        ],
    ]);
    

<a name="rule-numeric"></a>

#### numeric

The field under validation must be numeric.

<a name="rule-present"></a>

#### present

The field under validation must be present in the input data but can be empty.

<a name="rule-regex"></a>

#### regex:*pattern*

The field under validation must match the given regular expression.

**Note:** When using the `regex` pattern, it may be necessary to specify rules in an array instead of using pipe delimiters, especially if the regular expression contains a pipe character.

<a name="rule-required"></a>

#### required

The field under validation must be present in the input data and not empty. A field is considered "empty" if one of the following conditions are true:

<div class="content-list">
  <ul>
    <li>
      The value is <code>null</code>.
    </li>
    <li>
      The value is an empty string.
    </li>
    <li>
      The value is an empty array or empty <code>Countable</code> object.
    </li>
    <li>
      The value is an uploaded file with no path.
    </li>
  </ul>
</div>

<a name="rule-required-if"></a>

#### required_if:*anotherfield*,*value*,...

The field under validation must be present and not empty if the *anotherfield* field is equal to any *value*.

<a name="rule-required-unless"></a>

#### required_unless:*anotherfield*,*value*,...

The field under validation must be present and not empty unless the *anotherfield* field is equal to any *value*.

<a name="rule-required-with"></a>

#### required_with:*foo*,*bar*,...

The field under validation must be present and not empty *only if* any of the other specified fields are present.

<a name="rule-required-with-all"></a>

#### required_with_all:*foo*,*bar*,...

The field under validation must be present and not empty *only if* all of the other specified fields are present.

<a name="rule-required-without"></a>

#### required_without:*foo*,*bar*,...

The field under validation must be present and not empty *only when* any of the other specified fields are not present.

<a name="rule-required-without-all"></a>

#### required_without_all:*foo*,*bar*,...

The field under validation must be present and not empty *only when* all of the other specified fields are not present.

<a name="rule-same"></a>

#### same:*field*

The given *field* must match the field under validation.

<a name="rule-size"></a>

#### size:*value*

The field under validation must have a size matching the given *value*. For string data, *value* corresponds to the number of characters. For numeric data, *value* corresponds to a given integer value. For an array, *size* corresponds to the `count` of the array. For files, *size* corresponds to the file size in kilobytes.

<a name="rule-string"></a>

#### string

The field under validation must be a string. If you would like to allow the field to also be `null`, you should assign the `nullable` rule to the field.

<a name="rule-timezone"></a>

#### timezone

The field under validation must be a valid timezone identifier according to the `timezone_identifiers_list` PHP function.

<a name="rule-unique"></a>

#### unique:*table*,*column*,*except*,*idColumn*

The field under validation must be unique in a given database table. If the `column` option is not specified, the field name will be used.

**Specifying A Custom Column Name:**

    'email' => 'unique:users,email_address'
    

**Custom Database Connection**

Occasionally, you may need to set a custom connection for database queries made by the Validator. As seen above, setting `unique:users` as a validation rule will use the default database connection to query the database. To override this, specify the connection and the table name using "dot" syntax:

    'email' => 'unique:connection.users,email_address'
    

**Forcing A Unique Rule To Ignore A Given ID:**

Sometimes, you may wish to ignore a given ID during the unique check. For example, consider an "update profile" screen that includes the user's name, e-mail address, and location. Of course, you will want to verify that the e-mail address is unique. However, if the user only changes the name field and not the e-mail field, you do not want a validation error to be thrown because the user is already the owner of the e-mail address.

To instruct the validator to ignore the user's ID, we'll use the `Rule` class to fluently define the rule. In this example, we'll also specify the validation rules as an array instead of using the `|` character to delimit the rules:

    use Illuminate\Validation\Rule;
    
    Validator::make($data, [
        'email' => [
            'required',
            Rule::unique('users')->ignore($user->id),
        ],
    ]);
    

If your table uses a primary key column name other than `id`, you may specify the name of the column when calling the `ignore` method:

    'email' => Rule::unique('users')->ignore($user->id, 'user_id')
    

**Agregar cláusulas *Where* adicionales:**

You may also specify additional query constraints by customizing the query using the `where` method. For example, let's add a constraint that verifies the `account_id` is `1`:

    'email' => Rule::unique('users')->where(function ($query) {
        return $query->where('account_id', 1);
    })
    

<a name="rule-url"></a>

#### url

The field under validation must be a valid URL.

<a name="conditionally-adding-rules"></a>

## Conditionally Adding Rules

#### Validating When Present

In some situations, you may wish to run validation checks against a field **only** if that field is present in the input array. To quickly accomplish this, add the `sometimes` rule to your rule list:

    $v = Validator::make($data, [
        'email' => 'sometimes|required|email',
    ]);
    

In the example above, the `email` field will only be validated if it is present in the `$data` array.

> {tip} If you are attempting to validate a field that should always be present but may be empty, check out [this note on optional fields](#a-note-on-optional-fields)

#### Complex Conditional Validation

Sometimes you may wish to add validation rules based on more complex conditional logic. For example, you may wish to require a given field only if another field has a greater value than 100. Or, you may need two fields to have a given value only when another field is present. Adding these validation rules doesn't have to be a pain. First, create a `Validator` instance with your *static rules* that never change:

    $v = Validator::make($data, [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);
    

Let's assume our web application is for game collectors. If a game collector registers with our application and they own more than 100 games, we want them to explain why they own so many games. For example, perhaps they run a game resale shop, or maybe they just enjoy collecting. To conditionally add this requirement, we can use the `sometimes` method on the `Validator` instance.

    $v->sometimes('reason', 'required|max:500', function ($input) {
        return $input->games >= 100;
    });
    

The first argument passed to the `sometimes` method is the name of the field we are conditionally validating. The second argument is the rules we want to add. If the `Closure` passed as the third argument returns `true`, the rules will be added. This method makes it a breeze to build complex conditional validations. You may even add conditional validations for several fields at once:

    $v->sometimes(['reason', 'cost'], 'required', function ($input) {
        return $input->games >= 100;
    });
    

> {tip} The `$input` parameter passed to your `Closure` will be an instance of `Illuminate\Support\Fluent` and may be used to access your input and files.

<a name="validating-arrays"></a>

## Validación de Arrays

Validating array based form input fields doesn't have to be a pain. You may use "dot notation" to validate attributes within an array. For example, if the incoming HTTP request contains a `photos[profile]` field, you may validate it like so:

    $validator = Validator::make($request->all(), [
        'photos.profile' => 'required|image',
    ]);
    

You may also validate each element of an array. For example, to validate that each e-mail in a given array input field is unique, you may do the following:

    $validator = Validator::make($request->all(), [
        'person.*.email' => 'email|unique:users',
        'person.*.first_name' => 'required_with:person.*.last_name',
    ]);
    

Likewise, you may use the `*` character when specifying your validation messages in your language files, making it a breeze to use a single validation message for array based fields:

    'custom' => [
        'person.*.email' => [
            'unique' => 'Each person must have a unique e-mail address',
        ]
    ],
    

<a name="custom-validation-rules"></a>

## Custom Validation Rules

<a name="using-rule-objects"></a>

### Using Rule Objects

Laravel provides a variety of helpful validation rules; however, you may wish to specify some of your own. One method of registering custom validation rules is using rule objects. To generate a new rule object, you may use the `make:rule` Artisan command. Let's use this command to generate a rule that verifies a string is uppercase. Laravel will place the new rule in the `app/Rules` directory:

    php artisan make:rule Uppercase
    

Once the rule has been created, we are ready to define its behavior. A rule object contains two methods: `passes` and `message`. The `passes` method receives the attribute value and name, and should return `true` or `false` depending on whether the attribute value is valid or not. The `message` method should return the validation error message that should be used when validation fails:

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
    

Of course, you may call the `trans` helper from your `message` method if you would like to return an error message from your translation files:

    /**
     * Get the validation error message.
     *
     * @return string
     */
    public function message()
    {
        return trans('validation.uppercase');
    }
    

Once the rule has been defined, you may attach it to a validator by passing an instance of the rule object with your other validation rules:

    use App\Rules\Uppercase;
    
    $request->validate([
        'name' => ['required', new Uppercase],
    ]);
    

<a name="using-extensions"></a>

### Using Extensions

Another method of registering custom validation rules is using the `extend` method on the `Validator` [facade](/docs/{{version}}/facades). Let's use this method within a [service provider](/docs/{{version}}/providers) to register a custom validation rule:

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
    

The custom validator Closure receives four arguments: the name of the `$attribute` being validated, the `$value` of the attribute, an array of `$parameters` passed to the rule, and the `Validator` instance.

You may also pass a class and method to the `extend` method instead of a Closure:

    Validator::extend('foo', 'FooValidator@validate');
    

#### Defining The Error Message

You will also need to define an error message for your custom rule. You can do so either using an inline custom message array or by adding an entry in the validation language file. This message should be placed in the first level of the array, not within the `custom` array, which is only for attribute-specific error messages:

    "foo" => "Your input was invalid!",
    
    "accepted" => "The :attribute must be accepted.",
    
    // The rest of the validation error messages...
    

When creating a custom validation rule, you may sometimes need to define custom place-holder replacements for error messages. You may do so by creating a custom Validator as described above then making a call to the `replacer` method on the `Validator` facade. You may do this within the `boot` method of a [service provider](/docs/{{version}}/providers):

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
    

#### Implicit Extensions

By default, when an attribute being validated is not present or contains an empty value as defined by the [`required`](#rule-required) rule, normal validation rules, including custom extensions, are not run. For example, the [`unique`](#rule-unique) rule will not be run against a `null` value:

    $rules = ['name' => 'unique'];
    
    $input = ['name' => null];
    
    Validator::make($input, $rules)->passes(); // true
    

For a rule to run even when an attribute is empty, the rule must imply that the attribute is required. To create such an "implicit" extension, use the `Validator::extendImplicit()` method:

    Validator::extendImplicit('foo', function ($attribute, $value, $parameters, $validator) {
        return $value == 'foo';
    });
    

> {note} An "implicit" extension only *implies* that the attribute is required. Whether it actually invalidates a missing or empty attribute is up to you.