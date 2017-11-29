# Eventos

- [Introducción](#introduction)
- [Registrar eventos & *listeners*](#registering-events-and-listeners) 
    - [Generar eventos & *listeners*](#generating-events-and-listeners)
    - [Registrar eventos manualmente](#manually-registering-events)
- [Definir eventos](#defining-events)
- [Definir *listeners*](#defining-listeners)
- [Colas de *listeners*](#queued-event-listeners) 
    - [Acceder a la cola manualmente](#manually-accessing-the-queue)
    - [Gestionar trabajos (*jobs*) fallidos](#handling-failed-jobs)
- [Disparar eventos](#dispatching-events)
- [Suscriptores de eventos](#event-subscribers) 
    - [Escribir suscriptores de eventos](#writing-event-subscribers)
    - [Registrar suscriptores de eventos](#registering-event-subscribers)

<a name="introduction"></a>

## Introducción

Los eventos de Laravel proveen una implementación *observer*, permitiendo suscribir y capturar varios eventos que ocurren en la aplicación. Las clases de eventos se almacenan normalmente en `app/Events`, mientras que sus *listeners* (escuchadores) se almacenan en `app/Listeners`. No se preocupe si no encuentra estos directorios en la aplicación, puesto que se crearán tan pronto como comience a generar eventos y *listeners* utilizando los comandos de Artisan.

Los eventos son una buena forma de desacoplar varios aspectos de la aplicación, puesto que un único evento puede tener varios *listeners* que no dependan de otros. Por ejemplo, se puede enviar una notificación de Slack a un usuario cada vez que se envía un pedido. En lugar de acoplar el procesamiento del pedido a las notificaciones de Slack, se puede simplemente lanzar un evento `OrderShipped`, el cual puede recibir un *listener* y transformarlo en una notificación de Slack.

<a name="registering-events-and-listeners"></a>

## Registrar eventos & *listeners*

El `EventServiceProvider` que incluye Laravel es un buen lugar para registrar los *listeners* de eventos de toda su aplicación. La propiedad `listen` contiene un *array* de todos los eventos (claves) y sus *listeners* (valores). Por supuesto, se pueden añadir tantos eventos al *array* como sea necesario. Por ejemplo, para añadir un evento `OrderShipped`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'App\Events\OrderShipped' => [
            'App\Listeners\SendShipmentNotification',
        ],
    ];
    

<a name="generating-events-and-listeners"></a>

### Generar eventos & *listeners*

Por supuesto, la creación de los archivos para eventos y *listeners* resulta tediosa. En su lugar, simplemente añade *listeners* y eventos al `EventServiceProvider` y ejecuta el comando `event:generate`. Este comando generará cualquier evento o *listener* listado en el `EventServiceProvider`. Por supuesto, los eventos y *listeners* que ya existan quedarán intactos:

    php artisan event:generate
    

<a name="manually-registering-events"></a>

### Registrar eventos manualmente

Normalmente, los eventos se registrarán a través del *array* `$listen` en `EventServiceProvider`; sin embargo, se puede registrar eventos basados en *Closures* de forma manual en el método `boot` del `EventServiceProvider`:

    /**
     * Register any other events for your application.
     *
     * @return void
     */
    public function boot()
    {
        parent::boot();
    
        Event::listen('event.name', function ($foo, $bar) {
            //
        });
    }
    

#### Comodín de captura de eventos

Se pueden registrar *listeners* utilizando `*` como parámetro comodín, permitiendo capturar varios eventos en el mismo *listener*. Los *listeners* con comodín reciben el nombre del evento como primer parámetro y el *array* de datos del array como segundo argumento:

    Event::listen('event.*', function ($eventName, array $data) {
        //
    });
    

<a name="defining-events"></a>

## Definir eventos

Una clase de evento es simplemente un contenedor de datos que incluye información relacionada con el evento. Por ejemplo, asuma que el evento `OrderShipped` generado recibe un objeto [Eloquent](/docs/{{version}}/eloquent):

    <?php
    
    namespace App\Events;
    
    use App\Order;
    use Illuminate\Queue\SerializesModels;
    
    class OrderShipped
    {
        use SerializesModels;
    
        public $order;
    
        /**
         * Create a new event instance.
         *
         * @param  Order  $order
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }
    }
    

Como puede observar, el evento no contiene lógica. Es un contenedor simple para la instancia de `Order` que se compró. El *trait* `SerializesModels` utilizado por el evento *serializará* cualquier modelo Eloquent si el objeto del evento se *serializa* utilizando la función de PHP `serialize`.

<a name="defining-listeners"></a>

## Definir *listeners*

A continuación, echemos un vistazo al *listener* del evento de ejemplo. Los *listeners* reciben la instancia del evento en el método `handle`. El comando `event:generate` importará automáticamente la clase adecuada y incluirán el *type-hint* del evento en el método `handle`. En el método `handle` se puede ejecutar cualquier acción necesaria para responder al evento:

    <?php
    
    namespace App\Listeners;
    
    use App\Events\OrderShipped;
    
    class SendShipmentNotification
    {
        /**
         * Create the event listener.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }
    
        /**
         * Handle the event.
         *
         * @param  OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            // Access the order using $event->order...
        }
    }
    

> {tip} Los *listeners* de eventos pueden además incluir *type-hints* de cualquier dependencia que necesiten en sus constructores. Todos los *listeners* se resuelven a través del [service container](/docs/{{version}}/container) de Laravel, por lo que las dependencias se inyectarán automáticamente.

#### Detener la propagación de un evento

A veces, es necesario detener la propagación de un evento a otros *listeners*. Se puede hacer retornando `false` desde el método `handle` del *listener*.

<a name="queued-event-listeners"></a>

## Colas de *listeners*

Añadir la ejecución de un *listener* a una cola puede ser beneficioso si el *listener* va a ejecutar alguna tarea lenta como enviar un correo electrónico o peticiones HTTP. Antes de comenzar con colas de *listeners*, asegúrese de [configurar su cola](/docs/{{version}}/queues) y comenzar un *queue listener* en su servidor local o entorno de desarrollo.

Para especificar que un *listener* debe incluirse en una cola, debe añadir la interfaz `ShouldQueue` a la clase del mismo. Los *listeners* generados por el comando Artisan `event:generate` ya tienen esta interfaz importada en el *namespace* y pueden utilizarla directamente:

    <?php
    
    namespace App\Listeners;
    
    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;
    
    class SendShipmentNotification implements ShouldQueue
    {
        //
    }
    

¡Eso es todo! Ahora cuando se llame a un *listener* para un evento, se añadirá directamente a una cola por el *event dispatcher* (disparador de eventos) utilizando el [sistema de colas](/docs/{{version}}/queues) (*queue system*) de Laravel. Si no se lanza ninguna excepción durante la ejecución del *listener*, el trabajo se eliminará de la cola una vez que haya concluido su procesamiento.

#### Personalizar la conexión de colas & nombre de cola

Si desea personalizar la conexión para la cola y el nombre que utiliza un *listener*, se pueden definir las propiedades `$connection` y `$queue` de la clase *listener*:

    <?php
    
    namespace App\Listeners;
    
    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;
    
    class SendShipmentNotification implements ShouldQueue
    {
        /**
         * The name of the connection the job should be sent to.
         *
         * @var string|null
         */
        public $connection = 'sqs';
    
        /**
         * The name of the queue the job should be sent to.
         *
         * @var string|null
         */
        public $queue = 'listeners';
    }
    

<a name="manually-accessing-the-queue"></a>

### Acceder a la cola manualmente

Si necesita acceder a los métodos `delete` y `release` de la cola de trabajo subyacente del *listener*, puede hacerlo utilizando el *trait* `Illuminate\Queue\InteractsWithQueue`. Este *trait* se importa por defecto en los *listeners* generados y proporciona acceso a estos métodos:

    <?php
    
    namespace App\Listeners;
    
    use App\Events\OrderShipped;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    
    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;
    
        /**
         * Handle the event.
         *
         * @param  \App\Events\OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            if (true) {
                $this->release(30);
            }
        }
    }
    

<a name="handling-failed-jobs"></a>

### Gestionar trabajos (*jobs*) fallidos

En ocasiones, las colas de *listeners* pueden fallar. Si un *listener* de una cola supera el número máximo de intentos definidos por el *queue worker*, se ejecutará el método `failed` del *listener*. El método `failed` recibe la instancia del evento y la excepción que causó el fallo:

    <?php
    
    namespace App\Listeners;
    
    use App\Events\OrderShipped;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    
    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;
    
        /**
         * Handle the event.
         *
         * @param  \App\Events\OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            //
        }
    
        /**
         * Handle a job failure.
         *
         * @param  \App\Events\OrderShipped  $event
         * @param  \Exception  $exception
         * @return void
         */
        public function failed(OrderShipped $event, $exception)
        {
            //
        }
    }
    

<a name="dispatching-events"></a>

## Disparar eventos

Para lanzar un evento, se puede pasar una instancia del evento al *helper* `event`. El *helper* disparará el evento a todos los *listeners* registrados. Puesto que el *helper* `event` está disponible globalmente, se puede llamar desde cualquier parte de la aplicación:

    <?php
    
    namespace App\Http\Controllers;
    
    use App\Order;
    use App\Events\OrderShipped;
    use App\Http\Controllers\Controller;
    
    class OrderController extends Controller
    {
        /**
         * Ship the given order.
         *
         * @param  int  $orderId
         * @return Response
         */
        public function ship($orderId)
        {
            $order = Order::findOrFail($orderId);
    
            // Order shipment logic...
    
            event(new OrderShipped($order));
        }
    }
    

> {tip} Cuando se ejecutan *tests*, puede ser útil afirmar (*assert*) que se dispararon ciertos eventos sin necesidad de ejecutar sus *listeners*. Los [helpers incluidos](/docs/{{version}}/mocking#event-fake) hacen esto muy sencillo.

<a name="event-subscribers"></a>

## Suscriptores de Eventos – *Subscribers*

<a name="writing-event-subscribers"></a>

### Escribir suscriptores de eventos

Los suscriptores de eventos son clases que pueden suscribirse a varios eventos desde la propia clase, permitiendo definir varios controladores de eventos en una misma clase. Los suscriptores deben definir un método `subscribe`, el cual recibirá una instancia de un *event dispatcher*. Se puede llamar al método `listen` en el *dispatcher* dado para registrar *listeners* (capturadores/escuchadores) de eventos:

    <?php
    
    namespace App\Listeners;
    
    class UserEventSubscriber
    {
        /**
         * Handle user login events.
         */
        public function onUserLogin($event) {}
    
        /**
         * Handle user logout events.
         */
        public function onUserLogout($event) {}
    
        /**
         * Register the listeners for the subscriber.
         *
         * @param  Illuminate\Events\Dispatcher  $events
         */
        public function subscribe($events)
        {
            $events->listen(
                'Illuminate\Auth\Events\Login',
                'App\Listeners\UserEventSubscriber@onUserLogin'
            );
    
            $events->listen(
                'Illuminate\Auth\Events\Logout',
                'App\Listeners\UserEventSubscriber@onUserLogout'
            );
        }
    
    }
    

<a name="registering-event-subscribers"></a>

### Registrar suscriptores de eventos

Tras escribir el *subscriber*, ya está listo para registrarlo con el *event dispatcher* (disparador de eventos). Puede registrar *subscribers* utilizando la propiedad `$subscribe` en `EventServiceProvider`. Por ejemplo, para añadir el `UserEventSubscriber` a la lista:

    <?php
    
    namespace App\Providers;
    
    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;
    
    class EventServiceProvider extends ServiceProvider
    {
        /**
         * The event listener mappings for the application.
         *
         * @var array
         */
        protected $listen = [
            //
        ];
    
        /**
         * The subscriber classes to register.
         *
         * @var array
         */
        protected $subscribe = [
            'App\Listeners\UserEventSubscriber',
        ];
    }