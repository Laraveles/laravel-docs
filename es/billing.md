# Laravel Cashier

- [Introducción](#introduction)
- [Configuración](#configuration) 
    - [Stripe](#stripe-configuration)
    - [Braintree](#braintree-configuration)
    - [Configuración de divisas](#currency-configuration)
- [Suscripciones](#subscriptions) 
    - [Crear suscripciones](#creating-subscriptions)
    - [Comprobar el estado de una suscripción](#checking-subscription-status)
    - [Cambiar planes](#changing-plans)
    - [Cuantía de la suscipción](#subscription-quantity)
    - [Impuestos de la suscripción](#subscription-taxes)
    - [Cancelar suscripciones](#cancelling-subscriptions)
    - [Reactivar suscripciones](#resuming-subscriptions)
    - [Actualizar tarjetas de crédito](#updating-credit-cards)
- [Periodos de prueba de suscripciones](#subscription-trials) 
    - [Con tarjeta de crédito](#with-credit-card-up-front)
    - [Sin tarjeta de crédito](#without-credit-card-up-front)
- [Gestionar *Stripe Webhooks*](#handling-stripe-webhooks) 
    - [Definir gestores de eventos para *webhooks*](#defining-webhook-event-handlers)
    - [Suscripciones fallidas](#handling-failed-subscriptions)
- [Gestionar *Braintree Webhooks*](#handling-braintree-webhooks) 
    - [Definir gestores de eventos para *webhooks*](#defining-braintree-webhook-event-handlers)
    - [Suscripciones fallidas](#handling-braintree-failed-subscriptions)
- [Cargos únicos](#single-charges)
- [Facturas](#invoices) 
    - [Generar facturas en PDF](#generating-invoice-pdfs)

<a name="introduction"></a>

## Introducción

Laravel Cashier provee una interfaz fluida y expresiva para gestionar los servicios de cobro por suscripción de [Stripe](https://stripe.com) y [Braintree](https://www.braintreepayments.com). Gestiona prácticamente todo lo referente al código de facturación de suscripciones que tanto se teme. Además de la gestión de suscripciones básicas, Cashier maneja cupones, cambios de suscripciones, "cuantías", periodos de gracia de cancelación e incluso genera facturas PDF.

> {note} Si únicamente se está aplicando cargos "por compra" y no se ofrece un servicio de suscripción recurrente, no se debería usar Cashier. En su lugar, utilizar los SDKs de Stripe y Braintree directamente.

<a name="configuration"></a>

## Configuración

<a name="stripe-configuration"></a>

### Stripe

#### Composer

Primero, añadir el paquete de Cashier para Stripe a las dependencias:

    composer require "laravel/cashier":"~7.0"
    

#### Migraciones de BD

Antes de utilizar Cashier, será necesario [preparar la base de datos](/docs/{{version}}/migrations). Se añadirán algunas columnas a la tabla `users` y se creará una nueva tabla `subscriptions` para almacenar todo lo relativo a las suscripciones de los clientes:

    Schema::table('users', function ($table) {
        $table->string('stripe_id')->nullable();
        $table->string('card_brand')->nullable();
        $table->string('card_last_four')->nullable();
        $table->timestamp('trial_ends_at')->nullable();
    });
    
    Schema::create('subscriptions', function ($table) {
        $table->increments('id');
        $table->integer('user_id');
        $table->string('name');
        $table->string('stripe_id');
        $table->string('stripe_plan');
        $table->integer('quantity');
        $table->timestamp('trial_ends_at')->nullable();
        $table->timestamp('ends_at')->nullable();
        $table->timestamps();
    });
    

Una vez que se ha creadas las migraciones, ejecutar el comando de Artisan `migrate`.

#### Modelo facturable – *billable*

A continuación, añadir el *trait* `Billable` a la definición del modelo. Este *trait* incluye varios métodos que permitirán realizar las tareas de facturación más comunes, como crear suscripciones, aplicar cupones y actualizar la información de una tarjeta de crédito:

    use Laravel\Cashier\Billable;
    
    class User extends Authenticatable
    {
        use Billable;
    }
    

#### Claves API

Finalmente hay que configurar las claves API de Stripe en el archivo de configuración `services.php`. Se pueden obtener desde el panel de control de Stripe:

    'stripe' => [
        'model'  => App\User::class,
        'key' => env('STRIPE_KEY'),
        'secret' => env('STRIPE_SECRET'),
    ],
    

<a name="braintree-configuration"></a>

### Braintree

#### Consideraciones

Para muchas operaciones, las implementaciones de Cashier para Braintree y Stripe son las mismas. Ambos servicios proveen facturación por suscripción con tarjeta de crédito, pero Braintree soporta además pagos via PayPal. Además, Braintree no cuenta con algunas de las características de Stripe. Es importante tener esto en cuenta a la hora de decidir si utilizar Stripe o Braintree:

<div class="content-list">
  <ul>
    <li>
      Braintree soporta PayPal, Stripe no.
    </li>
    <li>
      Braintree no soporta los métodos <code>increment</code> y <code>decrement</code> en las suscripciones. Es una limitación de Braintree, no de Cashier.
    </li>
    <li>
      Braintree no soporta descuentos basados en porcentajes. De nuevo una limitación de Braintree, no de Cashier.
    </li>
  </ul>
</div>

#### Composer

Primero, añadir el paquete de Cashier para Braintree a las dependencias:

    composer require "laravel/cashier-braintree":"~2.0"
    

#### Service Provider

A continuación, registrar el archivo `Laravel\Cashier\CashierServiceProvider` como [service provider](/docs/{{version}}/providers) en el archivo de configuración `config/app.php`:

    Laravel\Cashier\CashierServiceProvider::class
    

#### Plan Credit Coupon

Antes de utilizar Cashier con Braintree, es necesario definir un descuento `plan-credit` en el panel de control de Braintree. Este descuento se utilizará para prorratear las suscripciones que cambian desde la suscripción anual a la mensual o viceversa.

La cantidad configurada en el panel de control de Braintree puede ser cualquier valor, ya que Cashier reemplazará la cantidad definida con la cantidad configurada cada vez que se aplique el cupón. Este cupón es necesario ya que Braintree no soporta de forma nativa el prorrateo de suscripciones entre frecuencias de suscripción.

#### Migraciones de BD

Antes de usar Cashier, será necesario [preparar la base de datos](/docs/{{version}}/migrations). Se añadirán algunas columnas a la tabla `users` y se creará una nueva tabla `subscriptions` para almacenar todo lo relativo a las suscripciones de los clientes:

    Schema::table('users', function ($table) {
        $table->string('braintree_id')->nullable();
        $table->string('paypal_email')->nullable();
        $table->string('card_brand')->nullable();
        $table->string('card_last_four')->nullable();
        $table->timestamp('trial_ends_at')->nullable();
    });
    
    Schema::create('subscriptions', function ($table) {
        $table->increments('id');
        $table->integer('user_id');
        $table->string('name');
        $table->string('braintree_id');
        $table->string('braintree_plan');
        $table->integer('quantity');
        $table->timestamp('trial_ends_at')->nullable();
        $table->timestamp('ends_at')->nullable();
        $table->timestamps();
    });
    

Una vez que se han creado las migraciones, simplemente ejecutar el comando de Artisan `migrate`.

#### Modelo facturable – *billable*

A continuación, añadir el *trait* `Billable` a la definición del modelo:

    use Laravel\Cashier\Billable;
    
    class User extends Authenticatable
    {
        use Billable;
    }
    

#### Claves API

Next, You should configure the following options in your `services.php` file:

    'braintree' => [
        'model'  => App\User::class,
        'environment' => env('BRAINTREE_ENV'),
        'merchant_id' => env('BRAINTREE_MERCHANT_ID'),
        'public_key' => env('BRAINTREE_PUBLIC_KEY'),
        'private_key' => env('BRAINTREE_PRIVATE_KEY'),
    ],
    

Ahora hay que añadir las siguientes llamadas al SDK de Braintree al método `boot` del `AppServiceProvider`:

    \Braintree_Configuration::environment(config('services.braintree.environment'));
    \Braintree_Configuration::merchantId(config('services.braintree.merchant_id'));
    \Braintree_Configuration::publicKey(config('services.braintree.public_key'));
    \Braintree_Configuration::privateKey(config('services.braintree.private_key'));
    

<a name="currency-configuration"></a>

### Configuración de divisas

La moneda por defecto de Cashier son dólares Estadounidenses (USD). Se puede cambiar la moneda por defecto llamando al método `Cashier::useCurrency` desde el método `boot` de uno de los *service providers*. El método `useCurrency` acepta dos cadenas como parámetros: la moneda y su símbolo:

    use Laravel\Cashier\Cashier;
    
    Cashier::useCurrency('eur', '€');
    

<a name="subscriptions"></a>

## Suscripciones

<a name="creating-subscriptions"></a>

### Crear suscripciones

Para crear una suscripción, es necesaria una instancia de un modelo *Billable*, que normalmente será una instancia de `App\User`. Una vez que se ha recuperado la instancia del modelo, se pude utilizar el método `newSubscription` para crear la suscripción del modelo:

    $user = User::find(1);
    
    $user->newSubscription('main', 'premium')->create($stripeToken);
    

El primer argumento pasado al método `newSubscription` corresponde al nombre de la suscripción. Si la aplicación únicamente ofrece una sola suscripción, se pude llamar a este método `main` o `primary`. El segundo argumento es el plan específico de Stripe/Braintree al que se está suscribiendo el usuario. Este valor debe corresponder con el identificador del plan en Stripe o Braintree.

El método `create`, el cual acepta el token de una tarjeta de crédito/fondo de Stripe, comenzará la suscripción así como actualizará la base de datos con el ID del cliente y otra información relevante de facturación.

#### Datos de usuario adicionales

Para especificar información adicional sobre el cliente, se puede pasar como segundo argumento al método `create`:

    $user->newSubscription('main', 'monthly')->create($stripeToken, [
        'email' => $email,
    ]);
    

Para saber más sobre los campos adicionales que soporta Stripe o Braintree, comprobar la [documentación sobre creación de clientes](https://stripe.com/docs/api#create_customer) de Stripe o la sección correspondiente de [Braintree](https://developers.braintreepayments.com/reference/request/customer/create/php).

#### Cupones

Para aplicar cupones al crear suscripciones, se puede utilizar el método `withCoupon`:

    $user->newSubscription('main', 'monthly')
         ->withCoupon('code')
         ->create($stripeToken);
    

<a name="checking-subscription-status"></a>

### Comprobar el estado de una suscripción

Una vez que un usuario se ha suscrito a la aplicación, se puede comprobar el estado de esta suscripción de varias formas. En primer lugar, el método `subscribed` retornará `true` si el usuario cuenta con una suscripción activa, incluso si el usuario se encuentra en el periodo de prueba:

    if ($user->subscribed('main')) {
        //
    }
    

El método `subscribed` también es un gran candidato para participar en los [middleware de ruta](/docs/{{version}}/middleware), permitiendo filtrar el acceso a las rutas y controladores basadas en el estado de las suscripciones de los usuarios:

    public function handle($request, Closure $next)
    {
        if ($request->user() && ! $request->user()->subscribed('main')) {
            // This user is not a paying customer...
            return redirect('billing');
        }
    
        return $next($request);
    }
    

Para determinar si un usuario se encuentra todavía en el periodo de prueba, utilizar el método `onTrial`. Este método puede resultar útil para mostrar un mensaje de advertencia al usuario de que se encuentra aún en este periodo:

    if ($user->subscription('main')->onTrial()) {
        //
    }
    

El método `suscribedToPlan` se puede utilizar para determinar si el usuario está suscrito a un plan concreto de Stripe/Braintree basado en el ID del plan. En este ejemplo, se determinará si la suscripción `main` del usuario está activamente suscrita al plan `monthly`:

    if ($user->subscribedToPlan('monthly', 'main')) {
        //
    }
    

#### Estado de suscripción cancelada

Para saber si un usuario fue alguna vez un usuario suscrito, pero ha cancelado su suscripción, existe el método `cancelled`:

    if ($user->subscription('main')->cancelled()) {
        //
    }
    

Se puede comprobar si el usuario ha cancelado la suscripción, pero se encuentra todavía en su "periodo de gracia" hasta que la suscripción caduque completamente. Por ejemplo, si un usuario cancela la suscripción el 5 de marzo y estaba prevista a caducar el 10 de marzo, el usuario se encuentra en el "periodo de gracia" hasta el 10 de marzo. Note that the `subscribed` method still returns `true` during this time:

    if ($user->subscription('main')->onGracePeriod()) {
        //
    }
    

<a name="changing-plans"></a>

### Cambiar planes

Después de que un usuario se suscriba a la aplicación, puede ocasionalmente requerir un cambio a otro plan de suscripción nuevo. Para cambiar un usuario a otra suscripción, pasar el identificador del plan al método `swap`:

    $user = App\User::find(1);
    
    $user->subscription('main')->swap('provider-plan-id');
    

Si el usuario está en periodo de prueba, se mantendrá. Además, si existe una "cuantía" para la suscripción, esa cantidad se mantendrá.

Para cambiar un plan y además cancelar el periodo de prueba en el que se encuentre el usuario, se puede usar el método `skipTrial`:

    $user->subscription('main')
            ->skipTrial()
            ->swap('provider-plan-id');
    

<a name="subscription-quantity"></a>

### Cuantía de la suscipción

> {note} Las cantidades de suscripción solo están soportadas en la versión de Stripe de Cashier. Braintree no tiene una característica que corresponda con "cantidad" como Stripe.

A veces las suscripciones dependen de la "cuantía". Por ejemplo, la aplicación puede cargar $10 al mes **por usuario** en una cuenta. Para incrementar o disminuir la cuantía de la suscripción, utilizar los métodos `incrementQuantity` y `decrementQuantity`:

    $user = User::find(1);
    
    $user->subscription('main')->incrementQuantity();
    
    // Add five to the subscription's current quantity...
    $user->subscription('main')->incrementQuantity(5);
    
    $user->subscription('main')->decrementQuantity();
    
    // Subtract five to the subscription's current quantity...
    $user->subscription('main')->decrementQuantity(5);
    

Alternativamente, se puede establecer una cuantía específica utilizando el método `updateQuantity`:

    $user->subscription('main')->updateQuantity(10);
    

El método `noProrate` se puede utilizar para actualizar la cuantía de la suscripción sin prorratear los cargos:

    $user->subscription('main')->noProrate()->updateQuantity(10);
    

Para más información sobre las cuantías de suscripción, consultar la [documentación de Stripe](https://stripe.com/docs/subscriptions/quantities).

<a name="subscription-taxes"></a>

### Impuestos de la suscripción

Para definir el porcentaje de impuestos que paga un usuario en una suscripción, utilizar el método `taxPercentage` del modelo *billable* y retornar un valor numérico entre 0 y 100 con no mas de 2 decimales.

    public function taxPercentage() {
        return 20;
    }
    

El método `taxPercentage` permite sumar impuestos en una base modelo-a-modelo, la cual puede ser útil para usuarios que abarcan varios países y reglas de impuestos.

> {note} El método `taxPercentage` aplica únicamente a los cargos por suscripción. Si se utiliza Cashier para "pagos únicos", será necesario especificar los impuestos en el momento del cobro.

<a name="cancelling-subscriptions"></a>

### Cancelar suscripciones

Para cancelar una suscripción, simplemente utilizar el método `cancel` sobre la suscripción del usuario:

    $user->subscription('main')->cancel();
    

Cuando se cancela una suscripción, Cashier establece el valor de `ends_at` de forma automática en la base de datos. Esta columna se utiliza para saber cuando el método `subscribed` debe devolver `false`. Por ejemplo, si un usuario cancela una suscripción el 1 de marzo, pero la suscripción no estaba programada para terminar hasta el 5 de marzo, el método `subscribed` continuaría retornando `true` hasta el 5 de marzo.

Se puede saber si un usuario ha cancelado su suscripción pero aun está en el "periodo de gracia" utilizando el método `onGracePeriod`:

    if ($user->subscription('main')->onGracePeriod()) {
        //
    }
    

If you wish to cancel a subscription immediately, call the `cancelNow` method on the user's subscription:

    $user->subscription('main')->cancelNow();
    

<a name="resuming-subscriptions"></a>

### Reactivar suscripciones

If a user has cancelled their subscription and you wish to resume it, use the `resume` method. The user **must** still be on their grace period in order to resume a subscription:

    $user->subscription('main')->resume();
    

Si el usuario cancela una suscripción y luego la reanuda antes de que haya caducado completamente, no será facturado inmediatamente. En cambio, la suscripción será simplemente reactivada y será facturado en su ciclo de facturación original.

<a name="updating-credit-cards"></a>

### Actualizar tarjetas de crédito

The `updateCard` method may be used to update a customer's credit card information. This method accepts a Stripe token and will assign the new credit card as the default billing source:

    $user->updateCard($stripeToken);
    

<a name="subscription-trials"></a>

## Periodos de prueba de suscripciones

<a name="with-credit-card-up-front"></a>

### Con tarjeta de crédito

If you would like to offer trial periods to your customers while still collecting payment method information up front, You should use the `trialDays` method when creating your subscriptions:

    $user = User::find(1);
    
    $user->newSubscription('main', 'monthly')
                ->trialDays(10)
                ->create($stripeToken);
    

This method will set the trial period ending date on the subscription record within the database, as well as instruct Stripe / Braintree to not begin billing the customer until after this date.

> {note} If the customer's subscription is not cancelled before the trial ending date they will be charged as soon as the trial expires, so you should be sure to notify your users of their trial ending date.

You may determine if the user is within their trial period using either the `onTrial` method of the user instance, or the `onTrial` method of the subscription instance. The two examples below are identical:

    if ($user->onTrial('main')) {
        //
    }
    
    if ($user->subscription('main')->onTrial()) {
        //
    }
    

<a name="without-credit-card-up-front"></a>

### Sin tarjeta de crédito

Si se desea ofrecer periodos de prueba sin almacenar previamente el método de pago del usuario, simplemente establecer el valor de la columna `trial_ends_at` en el registro del usuario a la fecha de finalización deseada. Esto se hace normalmente en el proceso de registro:

    $user = User::create([
        // Populate other user properties...
        'trial_ends_at' => Carbon::now()->addDays(10),
    ]);
    

> {note} Asegúrese de añadir un [date mutator](/docs/{{version}}/eloquent-mutators#date-mutators) para `trial_ends_at` en la definición del modelo.

Cashier se refiere a este tipo de periodo de prueba como "periodo de prueba genérico", puesto que no está asociado a ninguna suscripción. El método `onTrial` de la instancia `User` retornará `true` si la fecha actual no es posterior a `trial_ends_at`:

    if ($user->onTrial()) {
        // User is within their trial period...
    }
    

También se puede utilizar el método `onGenericTrial` si se desea conocer específicamente que el usuario está en el periodo de prueba "genérico" y no ha creado una suscripción todavía:

    if ($user->onGenericTrial()) {
        // User is within their "generic" trial period...
    }
    

Una vez que se está listo para crear una suscripción para el usuario, se puede utilizar el método `newSubscription` normalmente:

    $user = User::find(1);
    
    $user->newSubscription('main', 'monthly')->create($stripeToken);
    

<a name="handling-stripe-webhooks"></a>

## Gestionar *Stripe Webhooks*</h2> 

Ambos, Stripe y Braintree pueden notificar a la aplicación de una gran variedad de eventos a través de *webhooks*. Para gestionar los *Stripe webhooks*, definir una ruta que apunte al *webhook controller* de Cashier. Este controlador gestionará todas las peticiones entrantes de *webhooks* y las lanzará al método del controlador apropiado:

    Route::post(
        'stripe/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );
    

> {note} Una vez que se ha registrado la ruta, asegúrese de configurar la *webhook URL* en la configuración del panel de control de Stripe.

Por defecto, este controlador gestionará automáticamente la cancelación de suscripciones que tienen demasiados cargos fallidos (tal y como se defina en la configuración de Stripe); sin embargo, como pronto se descubrirá, se puede extender este controlador para gestionar cualquier evento *webhook* que sea necesario.

#### *Webhooks* & protección CSRF

Puesto que los *webhooks* de Stripe necesitan sortear la [protección CSRF](/docs/{{version}}/csrf) de Laravel, asegúrese de añadir la URI como una excepción en el *middleware* `VerifyCsrfToken` o listar la ruta fuera del grupo de *middleware* `web`:

    protected $except = [
        'stripe/*',
    ];
    

<a name="defining-webhook-event-handlers"></a>

### Definir gestores de eventos para *webhooks*

Cashier gestiona automáticamente la cancelación de suscripciones cuando hay cargos fallidos, pero si hay algún *webhook* de Stripe adicional que se desee gestionar, simplemente hay que extender el *WebhookController*. Los nombres de los métodos deben corresponder a la convención de Stripe, específicamente, los métodos deben contener el prefijo `handle` y utilizar la nomenclatura "camel case" del *Webhook* de Stripe a gestionar. Por ejemplo, para gestionar el *webhook* `invoice.payment_succeeded`, se debe añadir el método ` handleInvoicePaymentSucceeded` al controlador:

    <?php
    
    namespace App\Http\Controllers;
    
    use Laravel\Cashier\Http\Controllers\WebhookController as CashierController;
    
    class WebhookController extends CashierController
    {
        /**
         * Handle a Stripe webhook.
         *
         * @param  array  $payload
         * @return Response
         */
        public function handleInvoicePaymentSucceeded($payload)
        {
            // Handle The Event
        }
    }
    

<a name="handling-failed-subscriptions"></a>

### Suscripciones fallidas

¿Qué ocurre si expira la tarjeta de un cliente? Sin problema - Cashier incluye un *WebhookController* que cancela la suscripción del cliente. As noted above, all you need to do is point a route to the controller:

    Route::post(
        'stripe/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );
    

¡Eso es todo! Los cobros fallidos serán capturados y gestionados por el controlador. El controlador cancelará la suscripción del cliente cuando Stripe determine que la suscripción ha fallado (normalmente tras tres intentos de cobro fallidos).

<a name="handling-braintree-webhooks"></a>

## Gestionar *Braintree Webhooks*

Ambos, Stripe y Braintree pueden notificar a la aplicación de una gran variedad de eventos a través de *webhooks*. Para gestionar los *webhooks* de Braintree, defina una ruta que apunte al *webhook controller* de Cashier. Este controlador gestionará todas las peticiones entrantes de *webhooks* y las lanzará al método del controlador apropiado:

    Route::post(
        'braintree/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );
    

> {note} Una vez que se ha registrado la ruta, asegúrese de configurar la URL del *webhook* en la configuración del panel de control de Braintree.

Por defecto, este controlador gestionará automáticamente la cancelación de suscripciones que tienen demasiados cargos fallidos (tal y como se defina en la configuración de Braintree); sin embargo, como pronto descubrirá, se puede extender este controlador para gestionar cualquier evento *webhook* que sea necesario.

#### *Webhooks* & protección CSRF

Puesto que los *webhooks* de Braintree necesitan sortear la [protección CSRF](/docs/{{version}}/csrf) de Laravel, asegúrese de añadir la URI como una excepción en el *middleware* `VerifyCsrfToken` o listar la ruta fuera del grupo de *middleware* `web`:

    protected $except = [
        'braintree/*',
    ];
    

<a name="defining-braintree-webhook-event-handlers"></a>

### Definir gestores de eventos para *webhooks*

Cashier gestiona automáticamente la cancelación de suscripciones cuando hay cargos fallidos, pero si hay algún *webhook* de Stripe adicional que se desee gestionar, simplemente hay que extender el *WebhookController*. El nombre del método debe corresponderse con la convención de Cashier, los métodos deben contener el prefijo `handle` y en modo "camel case" el nombre del *webhook* de Braintree que desee gestionar. Por ejemplo, para gestionar el *webhook* `dispute_opened`, debe añadir el siguiente método al controlador `handleDisputeOpened`:

    <?php
    
    namespace App\Http\Controllers;
    
    use Braintree\WebhookNotification;
    use Laravel\Cashier\Http\Controllers\WebhookController as CashierController;
    
    class WebhookController extends CashierController
    {
        /**
         * Handle a Braintree webhook.
         *
         * @param  WebhookNotification  $webhook
         * @return Response
         */
        public function handleDisputeOpened(WebhookNotification $notification)
        {
            // Handle The Event
        }
    }
    

<a name="handling-braintree-failed-subscriptions"></a>

### Suscripciones fallidas

¿Qué pasa si se caduca una tarjeta de crédito? Cashier incluye un controlador de *Webhook* que cancelará automáticamente la suscripción. Simplemente hay que apuntar la ruta al controlador:

    Route::post(
        'braintree/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );
    

¡Eso es todo! Los cobros fallidos serán capturados y gestionados por el controlador. El controlador cancelará la suscripción del cliente cuando Braintree determine que la suscripción ha fallado (normalmente tras tres intentos fallidos de cobro). No olvidar: será necesario configurar la URI del *webhook* en el panel de control de Braintree.

<a name="single-charges"></a>

## Cargos únicos

### Cargo simple

> {note} Al utilizar Stripe, el método `charge` acepta la cantidad a cobrar en el **menor denominador de divisa utilizado por la aplicación**. Sin embargo, al utilizar Braintree, se debe pasar la cantidad exacta al método `charge`:

Para realizar un cargo "único" contra la tarjeta de crédito de un usuario suscrito, se puede utilizar el método `charge` de la instancia del modelo billable.

    // Stripe Accepts Charges In Cents...
    $user->charge(100);
    
    // Braintree Accepts Charges In Dollars...
    $user->charge(1);
    

El método `charge` acepta un *array* como segundo argumento, permitiendo pasar cualquier opción que se desee a la creación del cargo de Stripe/Braintree. Consultar la documentación de Stripe o Braintree para conocer las opciones disponibles en la creación de cargos:

    $user->charge(100, [
        'custom_option' => $value,
    ]);
    

El método `charge` lanzará una excepción si el cargo falla. Si el cargo es satisfactorio, se retornará la respuesta Stripe/Braintree completa:

    try {
        $response = $user->charge(100);
    } catch (Exception $e) {
        //
    }
    

### Cargo con factura

A veces es necesario hacer un cargo único y además generar una factura para el cargo para poder ofrecerle el recibo en PDF al cliente. El método `invoiceFor` permite hacer justo eso. Por ejemplo, se va a generar una factura para un cliente de $5.00 como "cargo único":

    // Stripe Accepts Charges In Cents...
    $user->invoiceFor('One Time Fee', 500);
    
    // Braintree Accepts Charges In Dollars...
    $user->invoiceFor('One Time Fee', 5);
    

La factura será cobrada inmediatamente contra la tarjeta de crédito del usuario. El método `invoiceFor` acepta además un *array* como tercer argumento, permitiendo pasar tantas opciones como sea necesario a la creación del cargo de Stripe/Braintree subyacente:

    $user->invoiceFor('One Time Fee', 500, [
        'custom-option' => $value,
    ]);
    

> {note} El método `invoiceFor` creará una factura de Stripe que re-intentará si se producen fallos de facturación. Si no se desea que las facturas re-intenten cobrar los cargos fallidos, es necesario cerrarlos utilizando el API de Stripe tras el primer cargo fallido.

<a name="invoices"></a>

## Facturas

Se puede obtener fácilmente un *array* de facturas del modelo billable utilizando el método `invoices`:

    $invoices = $user->invoices();
    
    // Include pending invoices in the results...
    $invoices = $user->invoicesIncludingPending();
    

Cuando se listan las facturas de un cliente, se pueden utilizar los métodos *helper* para mostrar la información relevante a la facturación. Por ejemplo, se puede listar cada factura en una tabla, permitiendo al usuario descargar cualquiera de ellas:

    <table>
        @foreach ($invoices as $invoice)
            <tr>
                <td>{{ $invoice->date()->toFormattedDateString() }}</td>
                <td>{{ $invoice->total() }}</td>
                <td><a href="/user/invoice/{{ $invoice->id }}">Download</a></td>
            </tr>
        @endforeach
    </table>
    

<a name="generating-invoice-pdfs"></a>

### Generar facturas en PDF

Desde una ruta o controlador, utilizar el método `downloadInvoice` para generar la descarga de una factura en PDF. Este método automáticamente genera la respuesta HTTP apropiada para enviar la descarga al navegador:

    use Illuminate\Http\Request;
    
    Route::get('user/invoice/{invoice}', function (Request $request, $invoiceId) {
        return $request->user()->downloadInvoice($invoiceId, [
            'vendor'  => 'Your Company',
            'product' => 'Your Product',
        ]);
    });