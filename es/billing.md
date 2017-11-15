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
    - [Defining Webhook Event Handlers](#defining-webhook-event-handlers)
    - [Suscripciones fallidas](#handling-failed-subscriptions)
- [Gestionar *Braintree Webhooks*](#handling-braintree-webhooks) 
    - [Defining Webhook Event Handlers](#defining-braintree-webhook-event-handlers)
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

#### Billable Model

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

Before using Cashier with Braintree, you will need to define a `plan-credit` discount in your Braintree control panel. This discount will be used to properly prorate subscriptions that change from yearly to monthly billing, or from monthly to yearly billing.

The discount amount configured in the Braintree control panel can be any value you wish, as Cashier will simply override the defined amount with our own custom amount each time we apply the coupon. This coupon is needed since Braintree does not natively support prorating subscriptions across subscription frequencies.

#### Migraciones de BD

Before using Cashier, we'll need to [prepare the database](/docs/{{version}}/migrations). Se añadirán algunas columnas a la tabla `users` y se creará una nueva tabla `subscriptions` para almacenar todo lo relativo a las suscripciones de los clientes:

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
    

Once the migrations have been created, simply run the `migrate` Artisan command.

#### Modelo facturable – *billable*

Next, add the `Billable` trait to your model definition:

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
    

Then you should add the following Braintree SDK calls to your `AppServiceProvider` service provider's `boot` method:

    \Braintree_Configuration::environment(config('services.braintree.environment'));
    \Braintree_Configuration::merchantId(config('services.braintree.merchant_id'));
    \Braintree_Configuration::publicKey(config('services.braintree.public_key'));
    \Braintree_Configuration::privateKey(config('services.braintree.private_key'));
    

<a name="currency-configuration"></a>

### Configuración de divisas

The default Cashier currency is United States Dollars (USD). You can change the default currency by calling the `Cashier::useCurrency` method from within the `boot` method of one of your service providers. The `useCurrency` method accepts two string parameters: the currency and the currency's symbol:

    use Laravel\Cashier\Cashier;
    
    Cashier::useCurrency('eur', '€');
    

<a name="subscriptions"></a>

## Suscripciones

<a name="creating-subscriptions"></a>

### Crear suscripciones

Para crear una suscripción, es necesaria una instancia de un modelo *Billable*, que normalmente será una instancia de `App\User`. Once you have retrieved the model instance, you may use the `newSubscription` method to create the model's subscription:

    $user = User::find(1);
    
    $user->newSubscription('main', 'premium')->create($stripeToken);
    

The first argument passed to the `newSubscription` method should be the name of the subscription. If your application only offers a single subscription, you might call this `main` or `primary`. The second argument is the specific Stripe / Braintree plan the user is subscribing to. This value should correspond to the plan's identifier in Stripe or Braintree.

The `create` method, which accepts a Stripe credit card / source token, will begin the subscription as well as update your database with the customer ID and other relevant billing information.

#### Datos de usuario adicionales

Para especificar información adicional sobre el cliente, se puede pasar como segundo argumento al método `create`:

    $user->newSubscription('main', 'monthly')->create($stripeToken, [
        'email' => $email,
    ]);
    

To learn more about the additional fields supported by Stripe or Braintree, check out Stripe's [documentation on customer creation](https://stripe.com/docs/api#create_customer) or the corresponding [Braintree documentation](https://developers.braintreepayments.com/reference/request/customer/create/php).

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
    

The `subscribedToPlan` method may be used to determine if the user is subscribed to a given plan based on a given Stripe / Braintree plan ID. In this example, we will determine if the user's `main` subscription is actively subscribed to the `monthly` plan:

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

Después de que un usuario se suscriba a la aplicación, puede ocasionalmente requerir un cambio a otro plan de suscripción nuevo. To swap a user to a new subscription, pass the plan's identifier to the `swap` method:

    $user = App\User::find(1);
    
    $user->subscription('main')->swap('provider-plan-id');
    

If the user is on trial, the trial period will be maintained. Also, if a "quantity" exists for the subscription, that quantity will also be maintained.

If you would like to swap plans and cancel any trial period the user is currently on, you may use the `skipTrial` method:

    $user->subscription('main')
            ->skipTrial()
            ->swap('provider-plan-id');
    

<a name="subscription-quantity"></a>

### Cuantía de la suscipción

> {note} Subscription quantities are only supported by the Stripe edition of Cashier. Braintree does not have a feature that corresponds to Stripe's "quantity".

Sometimes subscriptions are affected by "quantity". For example, your application might charge $10 per month **per user** on an account. To easily increment or decrement your subscription quantity, use the `incrementQuantity` and `decrementQuantity` methods:

    $user = User::find(1);
    
    $user->subscription('main')->incrementQuantity();
    
    // Add five to the subscription's current quantity...
    $user->subscription('main')->incrementQuantity(5);
    
    $user->subscription('main')->decrementQuantity();
    
    // Subtract five to the subscription's current quantity...
    $user->subscription('main')->decrementQuantity(5);
    

Alternativamente, se puede establecer una cuantía específica utilizando el método `updateQuantity`:

    $user->subscription('main')->updateQuantity(10);
    

The `noProrate` method may be used to update the subscription's quantity without pro-rating the charges:

    $user->subscription('main')->noProrate()->updateQuantity(10);
    

For more information on subscription quantities, consult the [Stripe documentation](https://stripe.com/docs/subscriptions/quantities).

<a name="subscription-taxes"></a>

### Impuestos de la suscripción

To specify the tax percentage a user pays on a subscription, implement the `taxPercentage` method on your billable model, and return a numeric value between 0 and 100, with no more than 2 decimal places.

    public function taxPercentage() {
        return 20;
    }
    

The `taxPercentage` method enables you to apply a tax rate on a model-by-model basis, which may be helpful for a user base that spans multiple countries and tax rates.

> {note} The `taxPercentage` method only applies to subscription charges. If you use Cashier to make "one off" charges, you will need to manually specify the tax rate at that time.

<a name="cancelling-subscriptions"></a>

### Cancelar suscripciones

Para cancelar una suscripción, simplemente utilizar el método `cancel` sobre la suscripción del usuario:

    $user->subscription('main')->cancel();
    

When a subscription is cancelled, Cashier will automatically set the `ends_at` column in your database. Esta columna se utiliza para saber cuando el método `subscribed` debe devolver `false`. Por ejemplo, si un usuario cancela una suscripción el 1 de marzo, pero la suscripción no estaba programada para terminar hasta el 5 de marzo, el método `subscribed` continuaría retornando `true` hasta el 5 de marzo.

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

If you would like to offer trial periods without collecting the user's payment method information up front, you may simply set the `trial_ends_at` column on the user record to your desired trial ending date. This is typically done during user registration:

    $user = User::create([
        // Populate other user properties...
        'trial_ends_at' => Carbon::now()->addDays(10),
    ]);
    

> {note} Be sure to add a [date mutator](/docs/{{version}}/eloquent-mutators#date-mutators) for `trial_ends_at` to your model definition.

Cashier refers to this type of trial as a "generic trial", since it is not attached to any existing subscription. The `onTrial` method on the `User` instance will return `true` if the current date is not past the value of `trial_ends_at`:

    if ($user->onTrial()) {
        // User is within their trial period...
    }
    

You may also use the `onGenericTrial` method if you wish to know specifically that the user is within their "generic" trial period and has not created an actual subscription yet:

    if ($user->onGenericTrial()) {
        // User is within their "generic" trial period...
    }
    

Once you are ready to create an actual subscription for the user, you may use the `newSubscription` method as usual:

    $user = User::find(1);
    
    $user->newSubscription('main', 'monthly')->create($stripeToken);
    

<a name="handling-stripe-webhooks"></a>

## Gestionar *Stripe Webhooks*</h2> 

Both Stripe and Braintree can notify your application of a variety of events via webhooks. To handle Stripe webhooks, define a route that points to Cashier's webhook controller. This controller will handle all incoming webhook requests and dispatch them to the proper controller method:

    Route::post(
        'stripe/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );
    

> {note} Once you have registered your route, be sure to configure the webhook URL in your Stripe control panel settings.

By default, this controller will automatically handle cancelling subscriptions that have too many failed charges (as defined by your Stripe settings); however, as we'll soon discover, you can extend this controller to handle any webhook event you like.

#### Webhooks & CSRF Protection

Since Stripe webhooks need to bypass Laravel's [CSRF protection](/docs/{{version}}/csrf), be sure to list the URI as an exception in your `VerifyCsrfToken` middleware or list the route outside of the `web` middleware group:

    protected $except = [
        'stripe/*',
    ];
    

<a name="defining-webhook-event-handlers"></a>

### Defining Webhook Event Handlers

Cashier automatically handles subscription cancellation on failed charges, but if you have additional Stripe webhook events you would like to handle, simply extend the Webhook controller. Los nombres de los métodos deben corresponder a la convención de Stripe, específicamente, los métodos deben contener el prefijo `handle` y utilizar la nomenclatura "camel case" del *Webhook* de Stripe a gestionar. For example, if you wish to handle the `invoice.payment_succeeded` webhook, you should add a `handleInvoicePaymentSucceeded` method to the controller:

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

What if a customer's credit card expires? No worries - Cashier includes a Webhook controller that can easily cancel the customer's subscription for you. As noted above, all you need to do is point a route to the controller:

    Route::post(
        'stripe/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );
    

¡Eso es todo! Los cobros fallidos serán capturados y gestionados por el controlador. El controlador cancelará la suscripción del cliente cuando Stripe determine que la suscripción ha fallado (normalmente tras tres intentos de cobro fallidos).

<a name="handling-braintree-webhooks"></a>

## Gestionar *Braintree Webhooks*

Both Stripe and Braintree can notify your application of a variety of events via webhooks. To handle Braintree webhooks, define a route that points to Cashier's webhook controller. This controller will handle all incoming webhook requests and dispatch them to the proper controller method:

    Route::post(
        'braintree/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );
    

> {note} Once you have registered your route, be sure to configure the webhook URL in your Braintree control panel settings.

By default, this controller will automatically handle cancelling subscriptions that have too many failed charges (as defined by your Braintree settings); however, as we'll soon discover, you can extend this controller to handle any webhook event you like.

#### Webhooks & CSRF Protection

Since Braintree webhooks need to bypass Laravel's [CSRF protection](/docs/{{version}}/csrf), be sure to list the URI as an exception in your `VerifyCsrfToken` middleware or list the route outside of the `web` middleware group:

    protected $except = [
        'braintree/*',
    ];
    

<a name="defining-braintree-webhook-event-handlers"></a>

### Defining Webhook Event Handlers

Cashier automatically handles subscription cancellation on failed charges, but if you have additional Braintree webhook events you would like to handle, simply extend the Webhook controller. Your method names should correspond to Cashier's expected convention, specifically, methods should be prefixed with `handle` and the "camel case" name of the Braintree webhook you wish to handle. For example, if you wish to handle the `dispute_opened` webhook, you should add a `handleDisputeOpened` method to the controller:

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
    

¡Eso es todo! Los cobros fallidos serán capturados y gestionados por el controlador. The controller will cancel the customer's subscription when Braintree determines the subscription has failed (normally after three failed payment attempts). Don't forget: you will need to configure the webhook URI in your Braintree control panel settings.

<a name="single-charges"></a>

## Cargos únicos

### Cargo simple

> {note} Al utilizar Stripe, el método `charge` acepta la cantidad a cobrar en el **menor denominador de divisa utilizado por la aplicación**. Sin embargo, al utilizar Braintree, se debe pasar la cantidad exacta al método `charge`:

Para realizar un cargo "único" contra la tarjeta de crédito de un usuario suscrito, se puede utilizar el método `charge` de la instancia del modelo billable.

    // Stripe Accepts Charges In Cents...
    $user->charge(100);
    
    // Braintree Accepts Charges In Dollars...
    $user->charge(1);
    

The `charge` method accepts an array as its second argument, allowing you to pass any options you wish to the underlying Stripe / Braintree charge creation. Consult the Stripe or Braintree documentation regarding the options available to you when creating charges:

    $user->charge(100, [
        'custom_option' => $value,
    ]);
    

The `charge` method will throw an exception if the charge fails. If the charge is successful, the full Stripe / Braintree response will be returned from the method:

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