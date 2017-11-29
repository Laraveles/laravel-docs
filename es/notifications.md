# Notificaciones

- [Introducción](#introduction)
- [Crear notificaciones](#creating-notifications)
- [Enviar notificaciones](#sending-notifications) 
    - [Uso del *trait Notifiable*](#using-the-notifiable-trait)
    - [Uso de la *facade Notification*](#using-the-notification-facade)
    - [Especificar de canales de entrega](#specifying-delivery-channels)
    - [Cola de notificaciones](#queueing-notifications)
    - [Notificaciones bajo demanda](#on-demand-notifications)
- [Notificación por correo electrónico](#mail-notifications) 
    - [Formatear mensajes de correo electrónico](#formatting-mail-messages)
    - [Personalizar el destinatario](#customizing-the-recipient)
    - [Personalizar el tema](#customizing-the-subject)
    - [Personalizar plantillas (*templates*)](#customizing-the-templates)
- [Notificaciones de correo Markdown](#markdown-mail-notifications) 
    - [Generación del mensaje](#generating-the-message)
    - [Construir el mensaje](#writing-the-message)
    - [Personalización de los componentes](#customizing-the-components)
- [Notificaciones de base de datos](#database-notifications) 
    - [Requisitos previos](#database-prerequisites)
    - [Formatear notificaciones de bases de datos](#formatting-database-notifications)
    - [Acceso a notificaciones](#accessing-the-notifications)
    - [Marcar notificaciones como leidas](#marking-notifications-as-read)
- [Transmitir notificaciones](#broadcast-notifications) 
    - [Requisitos previos](#broadcast-prerequisites)
    - [Formatear notificaciones de difusión (*broadcasting*)](#formatting-broadcast-notifications)
    - [Cómo escuchar notificaciones](#listening-for-notifications)
- [Notificaciones por SMS](#sms-notifications) 
    - [Requisitos previos](#sms-prerequisites)
    - [Formatear notificaciones SMS](#formatting-sms-notifications)
    - [Personalizar el número de origen](#customizing-the-from-number)
    - [Enrutar notificaciones SMS](#routing-sms-notifications)
- [Notificaciones de Slack](#slack-notifications) 
    - [Requisitos previos](#slack-prerequisites)
    - [Formatear notificaciones de Slack](#formatting-slack-notifications)
    - [Adjuntos en Slack](#slack-attachments)
    - [Enrutar notificaciones de Slack](#routing-slack-notifications)
- [Eventos de notificaciones](#notification-events)
- [Canales personalizados](#custom-channels)

<a name="introduction"></a>

## Introducción

Además del soporte para [enviar correo electrónico](/docs/{{version}}/mail), Laravel proporciona soporte para el envío de notificaciones a través de una variedad de canales de entrega, incluyendo correo, SMS (via [Nexmo](https://www.nexmo.com/)), y [Slack](https://slack.com). Las notificaciones también pueden almacenarse en una base de datos para que se muestren en su web.

Normalmente, las notificaciones deben ser breves, mensajes informativos que avisan a los usuarios de algo que ocurrió en su aplicación. Por ejemplo, si está escribiendo una aplicación de facturación, puede enviar una notificación de "factura pagada" a sus usuarios a través de los canales de correo electrónico y SMS.

<a name="creating-notifications"></a>

## Crear notificaciones

En Laravel, cada notificación está representada por una única clase (normalmente almacenada en el directorio `app/Notifications`). No hay que preocuparse si no se encuentra el directorio, se creará al ejecutar el comando de Artisan `make:notification`:

    php artisan make:notification InvoicePaid
    

Este comando colocará una nueva clase de notificación en el directorio `app/Notifications`. Cada clase de notificación contiene un método `via` y un número variable de métodos de creación de mensajes (como `toMail` o `toDatabase`) que convierten la notificación en un mensaje optimizado para ese canal en particular.

<a name="sending-notifications"></a>

## Enviar notificaciones

<a name="using-the-notifiable-trait"></a>

### Uso del *trait Notifiable*

Las notificaciones se pueden enviar de dos formas: usando el método `notify` del *trait* `Notifiable` o usando la *facade* `Notification`. Primero, exploremos usando el método del *trait*:

    <?php
    
    namespace App;
    
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    
    class User extends Authenticatable
    {
        use Notifiable;
    }
    

Este *trait* se incluye en el modelo `App\User` por defecto y contiene un método que puede usarse para enviar notificaciones: `notify`. El método `notify` espera recibir una instancia de una notificación:

    use App\Notifications\InvoicePaid;
    
    $user->notify(new InvoicePaid($invoice));
    

> {tip} Recuerde que puede utilizar el *trait* `Illuminate\Notifications\Notifiable` en cualquiera de sus modelos. No se limite a incluirlo únicamente en su modelo `User`.

<a name="using-the-notification-facade"></a>

### Uso de la *facade Notification*

Alternativamente, puede enviar notificaciones a través de la [facade](/docs/{{version}}/facades) `Notification`. Esto es útil principalmente cuando necesita enviar una notificación a varias entidades notificables, como una colección de usuarios. Para enviar notificaciones utilizando la *facade*, pase todas las entidades notificables y la instancia de notificación al método `send`:

    Notification::send($users, new InvoicePaid($invoice));
    

<a name="specifying-delivery-channels"></a>

### Especificación de canales de entrega

Cada clase de notificación tiene un método `via` que determina en qué canales se entregará la notificación. De serie, las notificaciones se pueden enviar a los canales `mail`, `database`, `broadcast`, `nexmo` y `slack`.

> {tip} Si desea utilizar otros canales de entrega como Telegram o Pusher, consulte el sitio web [Laravel Notification Channels website](http://laravel-notification-channels.com).

El método `via` recibe una instancia `$notifiable`, que será una instancia de la clase a la que se envía la notificación. Puede utilizar `$notifiable` para determinar en qué canales se debe entregar la notificación:

    /**
     * Get the notification's delivery channels.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function via($notifiable)
    {
        return $notifiable->prefers_sms ? ['nexmo'] : ['mail', 'database'];
    }
    

<a name="queueing-notifications"></a>

### Cola de notificaciones

> {note} Antes de hacer la cola de notificaciones, debe configurarla e [iniciar un *worker*](/docs/{{version}}/queues).

El envío de notificaciones puede tomar tiempo, especialmente si el canal necesita una llamada externa de una API para entregar la notificación. Para acelerar el tiempo de respuesta de su aplicación, deje que su notificación se coloque en la cola añadiendo la interfaz `ShouldQueue` y el *trait* `Queueable` a su clase. La interfaz y el *trait* ya se importan para todas las notificaciones generadas con `make:notification`, por lo que pueden añadirse inmediatamente a la clase de notificación:

    <?php
    
    namespace App\Notifications;
    
    use Illuminate\Bus\Queueable;
    use Illuminate\Notifications\Notification;
    use Illuminate\Contracts\Queue\ShouldQueue;
    
    class InvoicePaid extends Notification implements ShouldQueue
    {
        use Queueable;
    
        // ...
    }
    

Una vez que la interfaz `ShouldQueue` haya sido agregada, puede enviar la notificación normalmente. Laravel detectará la interfaz `ShouldQueue` en la clase y automáticamente pondrá en cola la entrega de la notificación:

    $user->notify(new InvoicePaid($invoice));
    

Si desea retrasar la entrega de la notificación, puede encadenar el método `delay` en su instancia:

    $when = Carbon::now()->addMinutes(10);
    
    $user->notify((new InvoicePaid($invoice))->delay($when));
    

<a name="on-demand-notifications"></a>

### Notificaciones bajo demanda

A veces, es posible que necesite enviar una notificación a alguien que no esté almacenado como "usuario" de su aplicación. Utilizando el método `Notification::route`, puede especificar la información de enrutamiento de la notificación *ad-hoc* antes de enviar la notificación:

    Notification::route('mail', 'taylor@laravel.com')
                ->route('nexmo', '5555555555')
                ->notify(new InvoicePaid($invoice));
    

<a name="mail-notifications"></a>

## Notificación por correo electrónico

<a name="formatting-mail-messages"></a>

### Formatear mensajes de correo electrónico

Si una notificación soporta que se envíe como un correo electrónico, debe definir un método `toMail` en la clase de notificación. Este método recibirá una entidad `$notifiable` y devolverá una instancia de `Illuminate\Notifications\Messages\MailMessage`. Los mensajes de correo pueden contener líneas de texto así como una "llamada a la acción". A continuación se muestra un ejemplo del método `toMail`:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        $url = url('/invoice/'.$this->invoice->id);
    
        return (new MailMessage)
                    ->greeting('Hello!')
                    ->line('One of your invoices has been paid!')
                    ->action('View Invoice', $url)
                    ->line('Thank you for using our application!');
    }
    

> {tip} Tenga en cuenta que se está usando `$this->invoice->id` en el método `toMail`. Puede pasar cualquier dato que necesite su notificación para generar su mensaje en el constructor.

En este ejemplo se registra un saludo, una línea de texto, una llamada a la acción, y luego otra línea de texto. Estos métodos proporcionados por el objeto `MailMessage` hacen que formatear pequeños correos electrónicos transaccionales sea fácil y rápido. El canal de correo traducirá los componentes del mensaje en una plantilla de correo electrónico HTML agradable y receptiva con una contraparte de texto plano. A continuación se muestra un ejemplo de un correo electrónico generado por el canal `mail`:

<img src="https://laravel.com/assets/img/notification-example.png" width="551" height="596" />

> {tip} Cuando envíe notificaciones por correo, asegúrese de establecer el valor `name` en su archivo de configuración `config/app.php`. Este valor se utilizará en el encabezado y pie de página de sus mensajes de notificación de correo.

#### Otras opciones de formato para las notificaciones

En lugar de definir las "líneas" del texto en la clase de notificación, puede utilizar el método `view` para especificar una plantilla personalizada que debería utilizarse para mostrar el correo electrónico de notificación:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)->view(
            'emails.name', ['invoice' => $this->invoice]
        );
    }
    

Además, puede devolver un objeto [mailable](/docs/{{version}}/mail) desde el método `toMail`:

    use App\Mail\InvoicePaid as Mailable;
    
    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return Mailable
     */
    public function toMail($notifiable)
    {
        return (new Mailable($this->invoice))->to($this->user->email);
    }
    

<a name="error-messages"></a>

#### Mensajes de error

Algunas notificaciones informan a los usuarios de errores, como el pago fallido de facturas. Puede indicar que un mensaje de correo electrónico se refiere a un error llamando al método `error` al crear su mensaje. Cuando se utiliza el método `error` en un mensaje de correo electrónico, el botón de llamada a la acción estará rojo en lugar de azul:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Message
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->error()
                    ->subject('Notification Subject')
                    ->line('...');
    }
    

<a name="customizing-the-recipient"></a>

### Personalizar el destinatario

Cuando envíe notificaciones a través del canal `mail`, el sistema de notificaciones buscará automáticamente una propiedad `email` en su entidad. Puede personalizar qué dirección de correo electrónico se utiliza para enviar la notificación definiendo un método `routeNotificationForMail` en la entidad:

    <?php
    
    namespace App;
    
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    
    class User extends Authenticatable
    {
        use Notifiable;
    
        /**
         * Route notifications for the mail channel.
         *
         * @return string
         */
        public function routeNotificationForMail()
        {
            return $this->email_address;
        }
    }
    

<a name="customizing-the-subject"></a>

### Personalizar el tema

De forma predeterminada, el asunto del correo electrónico es el nombre de la clase de la notificación con formato "title case". Por lo tanto, si su clase de notificación se llama `InvoicePaid`, el asunto del correo electrónico será `Invoice Paid`. Si desea especificar un asunto explícito para el mensaje, puede llamar al método `subject` al crear el mensaje:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->subject('Notification Subject')
                    ->line('...');
    }
    

<a name="customizing-the-templates"></a>

### Personalizar plantillas (*templates*)

Puede modificar la plantilla HTML y de texto plano utilizada en las notificaciones por correo electrónico publicando los recursos del paquete de notificación. Después de ejecutar este comando, las plantillas de notificación de correo se ubicarán en el directorio `resources/views/vendor/notifications`:

    php artisan vendor:publish --tag=laravel-notifications
    

<a name="markdown-mail-notifications"></a>

## Notificaciones de correo de Markdown

Las notificaciones de correo de Markdown le permiten aprovechar las plantillas prediseñadas de las notificaciones de correo electrónico, mientras que dan más libertad para escribir mensajes más largos y personalizados. Dado que los mensajes están escritos en Markdown, Laravel es capaz de generar plantillas HTML amigables para los mensajes, mientras que genera a su vez una contraparte de texto plano.

<a name="generating-the-message"></a>

### Generación del mensaje

Para generar una notificación con una plantilla de Markdown correspondiente, puede utilizar la opción `--markdown` del comando Artisan `make:notification`:

    php artisan make:notification InvoicePaid --markdown=mail.invoice.paid
    

Al igual que todas las demás notificaciones de correo, las notificaciones que utilizan plantillas de Markdown deben definir un método `toMail` en su clase. Sin embargo, en lugar de utilizar los métodos `line` y `action` para construir la notificación, utilice el método `markdown` para especificar el nombre de la plantilla Markdown que debe utilizarse:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        $url = url('/invoice/'.$this->invoice->id);
    
        return (new MailMessage)
                    ->subject('Invoice Paid')
                    ->markdown('mail.invoice.paid', ['url' => $url]);
    }
    

<a name="writing-the-message"></a>

### Construir el mensaje

Las notificaciones de correo de Markdown utilizan una combinación de componentes Blade y sintaxis de Markdown que le permiten construir fácilmente notificaciones aprovechando los componentes predefinidos de Laravel:

    @component('mail::message')
    # Invoice Paid
    
    Your invoice has been paid!
    
    @component('mail::button', ['url' => $url])
    View Invoice
    @endcomponent
    
    Thanks,<br>
    {{ config('app.name') }}
    @endcomponent
    

#### Componente *button*

El componente *button* muestra un enlace como oun botón centrado. El componente acepta dos argumentos, `url` y un `color` opcional. Los colores admitidos son azul, verde y rojo (`blue`, `green`, and `red`). Puede añadir tantos componentes *button* a una notificación como desee:

    @component('mail::button', ['url' => $url, 'color' => 'green'])
    View Invoice
    @endcomponent
    

#### Componente *panel*

El componente *panel* representa el bloque de texto dado en un panel que tiene un color de fondo ligeramente diferente al del resto de la notificación. Esto le permite llamar la atención sobre un bloque de texto dado:

    @component('mail::panel')
    This is the panel content.
    @endcomponent
    

#### Componente *table*

El componente *table* le permite transformar una tabla Markdown en una tabla HTML. El componente acepta la tabla de Markdown como contenido. La alineación de la columna de la tabla se soporta utilizando la sintaxis de alineación de la tabla Markdown por defecto:

    @component('mail::table')
    | Laravel       | Table         | Example  |
    | ------------- |:-------------:| --------:|
    | Col 2 is      | Centered      | $10      |
    | Col 3 is      | Right-Aligned | $20      |
    @endcomponent
    

<a name="customizing-the-components"></a>

### Personalización de los componentes

Puede exportar todos los componentes de notificación Markdown a su propia aplicación para personalizarlos. Para exportar los componentes, utilice el comando Artisan `vendor:publish` para publicar la etiqueta `laravel-mail`:

    php artisan vendor:publish --tag=laravel-mail
    

Este comando publicará los componentes de correo de Markdown en el directorio `resources/views/vendor/mail`. El directorio `mail` contendrá un directorio `html` y un directorio `markdown`, cada uno de los cuales contendrá sus respectivas representaciones de cada componente disponible. Es libre de personalizar estos componentes como quiera.

#### Personalizar el CSS

Después de exportar los componentes, el directorio `resources/views/vendor/mail/html/themes` contendrá un archivo `default.css`. Puede personalizar el CSS en este archivo y sus estilos estarán automáticamente incrustados dentro de las representaciones HTML de sus notificaciones Markdown.

> {tip} Si desea crear un tema completamente nuevo para los componentes de Markdown, simplemente escriba un nuevo archivo CSS dentro del directorio `html/themes` y cambie la opción `theme` de su archivo de configuración `mail`.

<a name="database-notifications"></a>

## Notificaciones de base de datos

<a name="database-prerequisites"></a>

### Requisitos previos

El canal de notificación `database` almacena la información en una tabla de base de datos. Esta tabla contendrá información como el tipo de notificación y los datos JSON personalizados que describen la notificación.

Puede consultar la tabla para mostrar las notificaciones en la interfaz de usuario de la aplicación. Pero, antes de que pueda hacerlo, necesitará crear una tabla de base de datos para guardar sus notificaciones. Puede utilizar el comando `notifications:table` para generar una migración con el esquema de tabla adecuado:

    php artisan notifications:table
    
    php artisan migrate
    

<a name="formatting-database-notifications"></a>

### Formatear notificaciones de bases de datos

Si una notificación soporta que se almacene en una tabla de la base de datos, debe definir un método `toDatabase` o `toArray` en la clase de notificación. Este método recibirá una entidad `$notifiable` y debería devolver un *array* PHP simple. El *array* devuelto se transformará a JSON y se guardará en la columna `data` de la tabla `notifications`. Veamos un ejemplo del método `toArray`:

    /**
     * Get the array representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function toArray($notifiable)
    {
        return [
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ];
    }
    

#### `toDatabase` versus `toArray`

El método `toArray` también se utiliza por el canal `broadcast` para determinar qué datos transmitir a su cliente JavaScript. Si desea tener dos representaciones de *array* diferentes para los canales `database` y `broadcast`, debe definir un método `toDatabase` en lugar de un método `toArray`.

<a name="accessing-the-notifications"></a>

### Acceso a notificaciones

Una vez que las notificaciones se almacenan en la base de datos, necesita una forma conveniente de acceder a ellas desde sus entidades *notifiables*. El *trait* `Illuminate\Notifications\Notifiable`, que se incluye en el modelo predeterminado de `App\User`, incluye una relación Eloquent con `notifications` que devuelve las notificaciones de la entidad. Para obtener notificaciones, puede acceder a este método como cualquier otra relación Eloquent. De forma predeterminada, las notificaciones se ordenarán por el campo de fecha `created_at`:

    $user = App\User::find(1);
    
    foreach ($user->notifications as $notification) {
        echo $notification->type;
    }
    

Si desea recuperar sólo las notificaciones "no leídas", puede utilizar la relación `unreadNotifications`. De nuevo, estas notificaciones se clasificarán por el campo de fecha `created_at`:

    $user = App\User::find(1);
    
    foreach ($user->unreadNotifications as $notification) {
        echo $notification->type;
    }
    

> {tip} Para acceder a las notificaciones desde su cliente JavaScript, debe definir un controlador para su aplicación que devuelva las notificaciones de una entidad *notifiable*, como el usuario actual. A continuación, puede realizar una solicitud HTTP a la URI de ese controlador desde su cliente JavaScript.

<a name="marking-notifications-as-read"></a>

### Marcar notificaciones como leidas

Normalmente, querrá marcar una notificación como "leída" cuando un usuario la vea. El *trait* `Illuminate\Notificaciones\Notificables` proporciona un método `markAsRead`, que actualiza la columna `read_at` en el registro de la base de datos de notificación:

    $user = App\User::find(1);
    
    foreach ($user->unreadNotifications as $notification) {
        $notification->markAsRead();
    }
    

Sin embargo, en lugar de pasar por cada notificación, puede utilizar el método `markAsRead` directamente en una colección de notificaciones:

    $user->unreadNotifications->markAsRead();
    

También puede utilizar una consulta de actualización en masa para marcar todas las notificaciones como leídas sin recuperarlas de la base de datos:

    $user = App\User::find(1);
    
    $user->unreadNotifications()->update(['read_at' => Carbon::now()]);
    

Por supuesto, puede borrar las notificaciones para eliminarlas completamente de la tabla:

    $user->notifications()->delete();
    

<a name="broadcast-notifications"></a>

## Transmitir notificaciones

<a name="broadcast-prerequisites"></a>

### Requisitos previos

Antes de hacer difusión (*broadcasting*) de las notificaciones, deberá configurar y familiarizarse con los servicios de [difusión de eventos](/docs/{{version}}/broadcasting) de Laravel. La difusión de eventos proporciona una forma de reaccionar a eventos lanzados por Laravel del lado del servidor desde su cliente JavaScript.

<a name="formatting-broadcast-notifications"></a>

### Formatear notificaciones de difusión (*broadcasting*)

El canal `broadcast` difunde notificaciones usando el servicio de [difusión de eventos](/docs/{{version}}/broadcasting) permitiendo a su cliente JavaScript responder a las notificaciones en tiempo real. Si una notificación soporta difusión (*broadcasting*), deberá definir un método `toBroadcast` en la clase de notificación. Este método recibirá una entidad `$notifiable` y deberá devolver una instancia de `BroadcastMessage`. Los datos devueltos serán codificados como JSON y se entregaran a tu cliente JavaScript. Veamos un ejemplo del método `toBroadcast`:

    use Illuminate\Notifications\Messages\BroadcastMessage;
    
    /**
     * Get the broadcastable representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return BroadcastMessage
     */
    public function toBroadcast($notifiable)
    {
        return new BroadcastMessage([
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ]);
    }
    

#### Configuración de la cola de difusión (*Broadcast Queue*)

Todas las notificaciones *broadcast* se colocan en una cola para su difusión. Si desea configurar la conexión de colas (*queue connection*) o el nombre de cola que se utiliza para la operación de *broadcast*, puede utilizar los métodos `onConnection` y `onQueue` de `BroadcastMessage`:

    return (new BroadcastMessage($data))
                    ->onConnection('sqs')
                    ->onQueue('broadcasts');
    

> {tip} Además de los datos que especifique, las notificaciones *broadcast* también contendrán un campo `type` que contendrá el nombre de clase de la notificación.

<a name="listening-for-notifications"></a>

### Cómo capturar notificaciones

Las notificaciones serán difundidas en un canal privado formateado usando la convención `{notifiable}.{id}`. Por tanto, si envía una notificación a una instancia de `App\User` con un ID de `1`, la notificación será difundida sobre el canal privado `App.User.1`. Cuando use [Laravel Echo](/docs/{{version}}/broadcasting) podrá escuchar fácilmente a las notificaciones en un canal usando el *helper* `notification`:

    Echo.private('App.User.' + userId)
        .notification((notification) => {
            console.log(notification.type);
        });
    

#### Personalizar el canal de notificación

Si desea personalizar los canales en los cuales una entidad *notifiable* recibirá sus notificaciones de difusión (*broadcast notifications*), deberá definir un método `receivesBroadcastNotificationsOn` en la entidad *notifiable*:

    <?php
    
    namespace App;
    
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    
    class User extends Authenticatable
    {
        use Notifiable;
    
        /**
         * The channels the user receives notification broadcasts on.
         *
         * @return string
         */
        public function receivesBroadcastNotificationsOn()
        {
            return 'users.'.$this->id;
        }
    }
    

<a name="sms-notifications"></a>

## Notificaciones por SMS

<a name="sms-prerequisites"></a>

### Requisitos previos

El envío de notificaciones SMS en Laravel se opera por [Nexmo](https://www.nexmo.com/). Antes de que pueda enviar notificaciones a través de Nexmo, necesita instalar el paquete de Composer `nexmo/client` y añadir algunas opciones de configuración a su archivo `config/services.php`. Puede copiar el ejemplo de configuración a continuación para comenzar:

    'nexmo' => [
        'key' => env('NEXMO_KEY'),
        'secret' => env('NEXMO_SECRET'),
        'sms_from' => '15556666666',
    ],
    

La opción `sms_from` es el número de teléfono desde el que se enviarán sus mensajes SMS. Debe generar un número de teléfono para su aplicación en el panel de control de Nexmo.

<a name="formatting-sms-notifications"></a>

### Formatear notificaciones SMS

Si una notificación soporta el ser enviada como SMS, debe definir un método `toNexmo` en la clase de notificación. Este método recibirá una entidad `$notifiable` y deberá devolver una instancia de `Illuminate\Notifications\Messages\NexmoMessage`:

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your SMS message content');
    }
    

#### Contenido Unicode

Si su mensaje SMS contiene caracteres unicode, debe llamar al método `unicode` al construir la instancia `NexmoMessage`:

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your unicode message')
                    ->unicode();
    }
    

<a name="customizing-the-from-number"></a>

### Personalizar el número de origen

Si desea enviar algunas notificaciones desde un número de teléfono diferente al especificado en su archivo `config/services.php`, puede utilizar el método `from` en una instancia `NexmoMessage`:

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your SMS message content')
                    ->from('15554443333');
    }
    

<a name="routing-sms-notifications"></a>

### Enrutar notificaciones SMS

Al enviar notificaciones a través del canal `nexmo`, el sistema de notificaciones buscará automáticamente un atributo `phone_number` en la entidad *notifiable*. Si desea personalizar el número de teléfono al que se envía la notificación, defina un método `routeNotificationForNexmo` en la entidad:

    <?php
    
    namespace App;
    
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    
    class User extends Authenticatable
    {
        use Notifiable;
    
        /**
         * Route notifications for the Nexmo channel.
         *
         * @return string
         */
        public function routeNotificationForNexmo()
        {
            return $this->phone;
        }
    }
    

<a name="slack-notifications"></a>

## Notificaciones de Slack

<a name="slack-prerequisites"></a>

### Requisitos previos

Antes de poder enviar notificaciones a través de Slack, debe instalar la librería *Guzzle HTTP* a través de Composer:

    composer require guzzlehttp/guzzle
    

También necesitará configurar una integración ["Incoming Webhook"](https://api.slack.com/incoming-webhooks) en su equipo de Slack. Esta integración le proporcionará una URL que puede utilizar para [erutar las notificaciones de Slack](#routing-slack-notifications).

<a name="formatting-slack-notifications"></a>

### Formatear notificaciones de Slack

Si una notificación soporta el envío de un mensaje a Slack, debe definir un método `toSlack` en la clase. Este método recibirá una entidad `$notifiable` y devolverá una instancia de `Illuminate\Notifications\Messages\MailMessage`. Los mensajes de Slack pueden contener texto así como un "archivo adjunto" que formatea texto adicional o una *array* de campos. Veamos un ejemplo básico de `toSlack`:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->content('One of your invoices has been paid!');
    }
    

En este ejemplo sólo estamos enviando una única línea de texto a Slack, que creará un mensaje parecido al siguiente:

<img src="https://laravel.com/assets/img/basic-slack-notification.png" />

#### Personalización del remitente & destinatario

Puede utilizar los métodos `from` y `to` para personalizar el remitente y el destinatario. El método `from` acepta un nombre de usuario e identificador emoji, mientras que el método `to` acepta un canal o nombre de usuario:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->from('Ghost', ':ghost:')
                    ->to('#other')
                    ->content('This will be sent to #other');
    }
    

También puede utilizar una imagen como su logotipo en lugar de un emoji:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->from('Laravel')
                    ->image('https://laravel.com/favicon.png')
                    ->content('This will display the Laravel logo next to the message');
    }
    

<a name="slack-attachments"></a>

### Adjuntos en Slack

También puede agregar "adjuntos" a los mensajes de Slack. Los archivos adjuntos proporcionan opciones de formato más completas que los mensajes de texto simples. En este ejemplo, enviaremos una notificación de error sobre una excepción ocurrida en una aplicación, incluyendo un enlace para ver más detalles sobre la excepción:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/exceptions/'.$this->exception->id);
    
        return (new SlackMessage)
                    ->error()
                    ->content('Whoops! Something went wrong.')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Exception: File Not Found', $url)
                                   ->content('File [background.jpg] was not found.');
                    });
    }
    

El ejemplo anterior generará un mensaje de Slack que se parecerá al siguiente:

<img src="https://laravel.com/assets/img/basic-slack-attachment.png" />

Los adjuntos también le permiten especificar un *array* de datos que deben presentarse al usuario. Los datos proporcionados se presentarán en un formato de tabla para facilitar la lectura:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/invoices/'.$this->invoice->id);
    
        return (new SlackMessage)
                    ->success()
                    ->content('One of your invoices has been paid!')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Invoice 1322', $url)
                                   ->fields([
                                        'Title' => 'Server Expenses',
                                        'Amount' => '$1,234',
                                        'Via' => 'American Express',
                                        'Was Overdue' => ':-1:',
                                    ]);
                    });
    }
    

El ejemplo anterior creará un mensaje de Slack que se parecerá al siguiente:

<img src="https://laravel.com/assets/img/slack-fields-attachment.png" />

#### Contenido *Markdown* adjunto

Si algunos de los campos de datos adjuntos contienen Markdown, puede utilizar el método `markdown` para indicar a Slack que analice y muestre los campos de datos adjuntos como texto con formato Markdown. Los valores aceptados por este método son: `pretext`, `text`, y / o `fields`. Para obtener más información sobre el formato de Slack para adjuntar archivos consulte la documentación [Slack API documentation](https://api.slack.com/docs/message-formatting#message_formatting):

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/exceptions/'.$this->exception->id);
    
        return (new SlackMessage)
                    ->error()
                    ->content('Whoops! Something went wrong.')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Exception: File Not Found', $url)
                                   ->content('File [background.jpg] was *not found*.')
                                   ->markdown(['text']);
                    });
    }
    

<a name="routing-slack-notifications"></a>

### Enrutar notificaciones de Slack

Para enrutar las notificaciones de Slack a la ubicación apropiada, defina un método `routeNotificationForSlack` en su entidad *notifiable*. Esto debería devolver la URL del *webhook* al que se debe enviar la notificación. Las URLs de *webhook* se pueden generar añadiendo un servicio "Incoming Webhook" a su equipo de Slack:

    <?php
    
    namespace App;
    
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    
    class User extends Authenticatable
    {
        use Notifiable;
    
        /**
         * Route notifications for the Slack channel.
         *
         * @return string
         */
        public function routeNotificationForSlack()
        {
            return $this->slack_webhook_url;
        }
    }
    

<a name="notification-events"></a>

## Eventos de notificación

Cuando se envía una notificación, el sistema de notificaciones activa el evento `Illuminate\Notifications\Events\NotificationSent`. Contiene la entidad "notifiable" y la propia instancia de notificación. Se pueden registrar escuchas para este evento en `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Notifications\Events\NotificationSent' => [
            'App\Listeners\LogNotification',
        ],
    ];
    

> {tip} Después de registrar a los escuchas en su `EventServiceProvider`, utilice el comando Artisan `event:generate` para generar rápidamente los *listeners* para estos eventos.

Dentro de un receptor de eventos, puede acceder a las propiedades `notifiable`, `notification`, y `channel` en el evento para obtener más información sobre el destinatario de la notificación o sobre la notificación en sí:

    /**
     * Handle the event.
     *
     * @param  NotificationSent  $event
     * @return void
     */
    public function handle(NotificationSent $event)
    {
        // $event->channel
        // $event->notifiable
        // $event->notification
    }
    

<a name="custom-channels"></a>

## Canal personalizado

Laravel incluye varios canales de notificación, pero posible programar sus propios *drivers* para entregar notificaciones a través de otros canales. Laravel lo hace sencillo. Para empezar, defina una clase que contenga un método `send`. El método debe recibir dos argumentos: `$notifiable` y `$notification`:

    <?php
    
    namespace App\Channels;
    
    use Illuminate\Notifications\Notification;
    
    class VoiceChannel
    {
        /**
         * Send the given notification.
         *
         * @param  mixed  $notifiable
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return void
         */
        public function send($notifiable, Notification $notification)
        {
            $message = $notification->toVoice($notifiable);
    
            // Send notification to the $notifiable instance...
        }
    }
    

Una vez que su clase de canal de notificación ha sido definida, simplemente puede devolver el nombre de la clase desde el método `via` de cualquiera de sus notificaciones:

    <?php
    
    namespace App\Notifications;
    
    use Illuminate\Bus\Queueable;
    use App\Channels\VoiceChannel;
    use App\Channels\Messages\VoiceMessage;
    use Illuminate\Notifications\Notification;
    use Illuminate\Contracts\Queue\ShouldQueue;
    
    class InvoicePaid extends Notification
    {
        use Queueable;
    
        /**
         * Get the notification channels.
         *
         * @param  mixed  $notifiable
         * @return array|string
         */
        public function via($notifiable)
        {
            return [VoiceChannel::class];
        }
    
        /**
         * Get the voice representation of the notification.
         *
         * @param  mixed  $notifiable
         * @return VoiceMessage
         */
        public function toVoice($notifiable)
        {
            // ...
        }
    }