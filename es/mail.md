# Correo

- [Introducción](#introduction) 
    - [Pre-requisitos del driver](#driver-prerequisites)
- [Generación de Mailables](#generating-mailables)
- [Escribiendo Mailables](#writing-mailables) 
    - [Configurando el Remitente](#configuring-the-sender)
    - [Configurando la Vista](#configuring-the-view)
    - [Ver datos](#view-data)
    - [Archivos Adjuntos](#attachments)
    - [Archivos Adjuntos en Línea](#inline-attachments)
    - [Customizing The SwiftMailer Message](#customizing-the-swiftmailer-message)
- [Archivos de Markdown](#markdown-mailables) 
    - [Generando Markdown Mailables](#generating-markdown-mailables)
    - [Escribiendo Mensajes Markdown](#writing-markdown-messages)
    - [Personalización de los componentes](#customizing-the-components)
- [Previewing Mailables In The Browser](#previewing-mailables-in-the-browser)
- [Enviar Correo](#sending-mail) 
    - [Cola de Correos](#queueing-mail)
- [Correo & Desarrollo Local](#mail-and-local-development)
- [Eventos](#events)

<a name="introduction"></a>

## Introducción

Laravel proporciona una API clara y simple sobre la popular librería [SwiftMailer](https://swiftmailer.symfony.com/) con drivers para SMTP, Mailgun, SparkPost, Amazon SES, para la función `mail` de PHP y `sendmail`, permitiéndole comenzar a enviar correos rápidamente a través de un servicio local o en la nube de su elección.

<a name="driver-prerequisites"></a>

### Pre-requisitos del driver

Los drivers basados en API's como Mailgun y Mandril son a menudo más simples y rápidos que los servidores SMTP. Siempre que sea posible, debe usar uno de estos drivers. Todos los drivers del API requieren la librería Guzzle HTTP, la cual puede instalarse a través del gestor de paquetes Composer:

    composer require guzzlehttp/guzzle
    

#### Driver Mailgun

Para utilizar el driver de Mailgun, se debe instalar primero Guzzle, y luego colocar `mailgun` en la opción de `driver` en el archivo de configuración ubicado en `config/mail.php`. A continuación, se verifica que en el archivo `config/services.php` se encuentren las siguientes opciones:

    'mailgun' => [
        'domain' => 'your-mailgun-domain',
        'secret' => 'your-mailgun-key',
    ],
    

#### Driver SparkPost

Para usar el driver SparkPost, instale primero Guzzle, luego configure la opción `driver` en el archivo de configuración `config/mail.php` con el valor `sparkpost`. A continuación, se verifica que en el archivo `config/services.php` se encuentren las siguientes opciones:

    'sparkpost' => [
        'secret' => 'your-sparkpost-key',
    ],
    

#### Driver SES

Para usar el driver Amazon SES, primero debe instalar el SDK Amazon AWS para PHP. Puede instalar esta librería añadiendo la siguiente línea a la sección `require` del archivo `composer.json` y ejecutando el comando `composer update`:

    "aws/aws-sdk-php": "~3.0"
    

A continuación, configure la opción `driver` en el archivo de configuración `config/mail.php` con el valor `ses` y verifique que el archivo de configuración `config/services.php` contiene las siguientes opciones:

    'ses' => [
        'key' => 'your-ses-key',
        'secret' => 'your-ses-secret',
        'region' => 'ses-region',  // e.g. us-east-1
    ],
    

<a name="generating-mailables"></a>

## Generación de Mailables

En Laravel, cada tipo de correo electrónico enviado por su aplicación se representa como una clase "mailable". Todas estas clases se almacenan en el directorio `app/Mail`. No se preocupe si no ve este directorio en su aplicación, se creará para usted cuando cree su primera clase mailable usando el comando `make:mail`:

    php artisan make:mail OrderShipped
    

<a name="writing-mailables"></a>

## Escribiendo Mailables

Toda la configuración de una clase mailable se realiza en el método `build`. Desde este método puede llamar a varios métodos como `from`, `subject`, `view` y `attach` para configurar la presentación y entrega del correo electrónico.

<a name="configuring-the-sender"></a>

### Configurando el Remitente

#### El método `from`

Primero, exploremos la configuración del remitente del correo electrónico. O, en otras palabras, quién se configurará en el método "from". Hay dos formas de configurar el remitente. La primera, puede usar el método `from` dentro del método `build` de su clase mailable:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->view('emails.orders.shipped');
    }
    

#### Usar una Dirección de Remitente Global con `from`

Sin embargo, si la aplicación utiliza la misma dirección "from" para todos los correos electrónicos, puede resultar engorroso llamar al método `from` en cada clase de mailable que se genere. En su lugar, se puede especificar una dirección "from" en el archivo de configuración `config/mail.php`. Esta dirección se usará si no se especifica ninguna otra dirección "from" dentro de la clase mailable:

    'from' => ['address' => 'example@example.com', 'name' => 'App Name'],
    

<a name="configuring-the-view"></a>

### Configurando la Vista

Dentro del método `build` de clase mailable, se puede usar el método `view` para especificar qué plantilla se debe usar para representar el contenido del correo electrónico. Dado que cada correo electrónico normalmente usa una [plantilla de Blade](/docs/{{version}}/blade) para representar el contenido, se cuenta con toda la potencia y la comodidad del motor de plantillas para construir el HTML del correo electrónico:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped');
    }
    

> {tip} Se puede crear un directorio `resources/views/emails` para guardar todas las vistas de correo electrónico; sin embargo, se pueden colocar donde se desee dentro del directorio `resources/views`.

#### Emails de texto plano

Si se desea definir una versión de texto sin formato de correo electrónico, se puede usar el método `text`. Al igual que el método `view`, el método `text` acepta un nombre de plantilla que se utilizará para representar el contenido del correo electrónico. Se es libre de definir una versión HTML y de texto sin formato del mensaje:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->text('emails.orders.shipped_plain');
    }
    

<a name="view-data"></a>

### Ver datos

#### Vía Propiedades Públicas

Por lo general, se querrá pasar algunos datos a una vista que se puede utilizar al representar el HTML de un correo electrónico. Hay dos maneras en que puede hacer que los datos estén disponibles para una vista. Primero, cualquier propiedad pública definida en la clase mailable se pondrá automáticamente a disposición de la vista. Entonces, por ejemplo, se puede pasar datos al constructor de la clase mailable y establecer esos datos a propiedades públicas definidas dentro de la clase:

    <?php
    
    namespace App\Mail;
    
    use App\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;
    
    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;
    
        /**
         * The order instance.
         *
         * @var Order
         */
        public $order;
    
        /**
         * Create a new message instance.
         *
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }
    
        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped');
        }
    }
    

Una vez que los datos se han establecido en una propiedad pública, estarán automáticamente disponibles en la vista, por lo que puede acceder a ella como si tuviera acceso a cualquier otro dato en las plantillas Blade:

    <div>
        Price: {{ $order->price }}
    </div>
    

#### Vía el método `with`:

Si se desea personalizar el formato de los datos del correo electrónico antes de enviarlo a la plantilla, se puede pasar manualmente los datos a la vista a través del método `with`. Por lo general, se pasarán datos a través del constructor de la clase mailable; sin embargo, para que los datos no estén disponibles automáticamente en la plantilla se debe establecer estos datos como propiedades `protected` ó `private`. Luego, cuando se llame al método `with`, se debe pasar un array de los datos que se desee poner a disposición de la plantilla:

    <?php
    
    namespace App\Mail;
    
    use App\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;
    
    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;
    
        /**
         * The order instance.
         *
         * @var Order
         */
        protected $order;
    
        /**
         * Create a new message instance.
         *
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }
    
        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->with([
                            'orderName' => $this->order->name,
                            'orderPrice' => $this->order->price,
                        ]);
        }
    }
    

Una vez que los datos se pasen con el método `with`, estarán automáticamente disponibles en la vista, por lo que puede acceder a ella como si tuviera acceso a cualquier otro dato en las plantillas Blade:

    <div>
        Price: {{ $orderPrice }}
    </div>
    

<a name="attachments"></a>

### Archivos Adjuntos

Para agregar archivos adjuntos a un correo electrónico, use el método `attach` dentro del método `build` de la clase mailable. El método `attach` acepta la ruta entera del archivo como primer argumento:

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attach('/path/to/file');
        }
    

Al adjuntar archivos a un mensaje, también se puede especificar el nombre de visualización y/o tipo MIME pasando un `array` como el segundo argumento del método `attach`:

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attach('/path/to/file', [
                            'as' => 'name.pdf',
                            'mime' => 'application/pdf',
                        ]);
        }
    

#### Raw Data Attachments

El método `attachData` se puede usar para adjuntar una cadena de bytes sin formato como un archivo adjunto. Por ejemplo, se puede usar este método si ha generado un PDF en la memoria y desea adjuntarlo al correo electrónico sin escribirlo en el disco. El método `attachData` acepta los bytes de datos brutos como su primer argumento, el nombre del archivo como su segundo argumento y un array de opciones como su tercer argumento:

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attachData($this->pdf, 'name.pdf', [
                            'mime' => 'application/pdf',
                        ]);
        }
    

<a name="inline-attachments"></a>

### Archivos Adjuntos en Línea

La incorporación de imágenes en línea en correos electrónicos suele ser un proceso engorroso; sin embargo, Laravel proporciona una manera práctica para adjuntar imágenes a los correos y recuperar el CID apropiado. Para incluir una imagen en linea, se utiliza el método `embed` de la variable `$message` en la vista del e-mail. Laravel automáticamente hace que la variable `$message` esté disponible para todas las plantillas de correo electrónico, por lo que no hay que preocuparse por pasarla manualmente:

    <body>
        Here is an image:
    
        <img src="{{ $message->embed($pathToFile) }}">
    </body>
    

> {note} La variable `$message` no está disponible en los mensajes de markdown.

#### Incrustar datos brutos adjuntos

Si se tienen datos sin procesar (raw data string) que se desean incrustar en una plantilla de correo electrónico, se puede utilizar el método `embedData` en la variable `$message`:

    <body>
        Here is an image from raw data:
    
        <img src="{{ $message->embedData($data, $name) }}">
    </body>
    

<a name="customizing-the-swiftmailer-message"></a>

### Personalizar el mensaje de SwiftMailer

El método `withSwiftMessage` de la clase base `Mailable` permite registrar un *callback* que se invocará con la instancia de mensaje de SwiftMailer sin procesar antes de enviar el mensaje. Esto le da la oportunidad de personalizar el mensaje antes de que se entregue:

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            $this->view('emails.orders.shipped');
    
            $this->withSwiftMessage(function ($message) {
                $message->getHeaders()
                        ->addTextHeader('Custom-Header', 'HeaderValue');
            });
        }
    

<a name="markdown-mailables"></a>

## Markdown Mailables

Los mensajes mailers de Markdown permiten aprovechar las plantillas y los componentes precompilados de las notificaciones por correo en los mails. Dado que los mensajes están escritos en Markdown, Laravel es capaz de generar hermosas plantillas HTML para los mensajes, mientras que también genera automáticamente una contraparte de texto plano.

<a name="generating-markdown-mailables"></a>

### Generando Markdown Mailables

Para generar una mailable con una plantilla de Markdown correspondiente, puede utilizar la opción `--markdown` del comando Artisan `make:mail`:

    php artisan make:mail OrderShipped --markdown=emails.orders.shipped
    

Luego, al configurar el mailable dentro de su método `build`, se llama al método `markdown` en lugar del método `view`. Los métodos `markdown` aceptan el nombre de la plantilla de Markdown y una array opcional de datos para poner a disposición de la plantilla:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->markdown('emails.orders.shipped');
    }
    

<a name="writing-markdown-messages"></a>

### Escribiendo Mensajes Markdown

Los mailables de Markdown utilizan una combinación de componentes Blade y sintaxis de Markdown que le permiten construir fácilmente mensaje de correo aprovechando los componentes predefinidos de Laravel:

    @component('mail::message')
    # Order Shipped
    
    Your order has been shipped!
    
    @component('mail::button', ['url' => $url])
    View Order
    @endcomponent
    
    Thanks,<br>
    {{ config('app.name') }}
    @endcomponent
    

> {tip} No usar sangría excesiva al escribir correos electrónicos de Markdown. Los analizadores de Markdown renderizarán contenido sangrado como bloques de código.

#### Componente *button*

El componente *button* muestra un enlace de botón centrado. El componente acepta dos argumentos, `url` y opcional `color`. Los colores admitidos son azul, verde y rojo (`blue`, `green`, and `red`). Puede añadir tantos componentes *button* a un mensaje como desee:

    @component('mail::button', ['url' => $url, 'color' => 'green'])
    View Order
    @endcomponent
    

#### Componente *panel*

El componente panel representa el bloque de texto dado en un panel que tiene un color de fondo ligeramente diferente que el resto del mensaje. Esto le permite llamar la atención sobre un bloque de texto dado:

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

Puede exportar todos los componentes de Mail Markdown a su propia aplicación para personalizarlos. Para exportar los componentes, utilice el comando Artisan `vendor:publish` para publicar la etiqueta `laravel-mail`:

    php artisan vendor:publish --tag=laravel-mail
    

Este comando publicará los componentes de correo de Markdown en el directorio `resources/views/vendor/mail`. El directorio `mail` contendrá un directorio `html` y un directorio `markdown`, cada uno de los cuales contendrá sus respectivas representaciones de cada componente disponible. Es libre de personalizar estos componentes como quiera.

#### Personalizar el CSS

Después de exportar los componentes, el directorio `resources/views/vendor/mail/html/themes` contendrá un archivo `default.css`. Puede personalizar el CSS en este archivo y sus estilos estarán automáticamente incrustados dentro de las representaciones HTML de sus mensajes de mail Markdown.

> {tip} Si desea crear un tema completamente nuevo para los componentes de Markdown, simplemente escriba un nuevo archivo CSS dentro del directorio `html/themes` y cambie la opción `theme` de su archivo de configuración `mail`.

<a name="previewing-mailables-in-the-browser"></a>

## Vista previa de Mailables en el navegador

Al diseñar una plantilla de mailable, es conveniente hacer una vista previa rápida del mial procesado en el navegador como una típica plantilla Blade. Por esta razón, Laravel permite devolver cualquier mailable directamente desde una ruta anónima o un controlador. Cuando se devuelve un mailable, se procesará y se mostrará en el navegador, lo que permite obtener una vista previa rápida de su diseño sin necesidad de enviarlo a una dirección de correo electrónico real:

    Route::get('/mailable', function () {
        $invoice = App\Invoice::find(1);
    
        return new App\Mail\InvoicePaid($invoice);
    });
    

<a name="sending-mail"></a>

## Enviar Correo

Para enviar un mensaje, se usa el metodo `to` en el `Mail` [facade](/docs/{{version}}/facades). El método `to` acepta una dirección de correo electrónico, una instancia de usuario o una colección de usuarios. Si se pasa un objeto o una colección de objetos, el remitente utilizará automáticamente las propiedades `email` y `name` al configurar los destinatarios del correo electrónico, así que hay que asegurarse de que estos atributos estén disponibles en los objetos. Una vez que haya especificado los destinatarios, se puede pasar una instancia de su clase mailable al método `send`:

    <?php
    
    namespace App\Http\Controllers;
    
    use App\Order;
    use App\Mail\OrderShipped;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Mail;
    use App\Http\Controllers\Controller;
    
    class OrderController extends Controller
    {
        /**
         * Ship the given order.
         *
         * @param  Request  $request
         * @param  int  $orderId
         * @return Response
         */
        public function ship(Request $request, $orderId)
        {
            $order = Order::findOrFail($orderId);
    
            // Ship order...
    
            Mail::to($request->user())->send(new OrderShipped($order));
        }
    }
    

Por supuesto, no está limitado a especificar los destinatarios "to" al enviar un mensaje. Se es libre de configurar los destinatarios "to", "cc" y "bcc", todo dentro de una única llamada a un método encadenado:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->send(new OrderShipped($order));
    

<a name="queueing-mail"></a>

### Cola de Correos

#### Encolar un Correo

El envío de correo electrónico puede alargar drásticamente el tiempo de respuesta de la aplicación, muchos desarrolladores eligen hacer una cola de los mensajes para ser enviados en segundo plano. Laravel lo hace fácil a través del uso de su [API de colas unificada](/docs/{{version}}/queues). Para poner en cola un mensaje de correo, se usa el método `queue` en la facade `Mail` después de especificar los destinatarios del mensaje:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue(new OrderShipped($order));
    

Este método se encargará automáticamente de empujar un trabajo en la cola para enviar el mensaje de correo en segundo plano. Por supuesto, hay que [configurar las colas](/docs/{{version}}/queues) antes de utilizar esta función.

#### Encolar Mensajes con Retraso

Si se desea retrasar la entrega de un correo electrónico en cola, se puede usar el método `later`. Como primer argumento, el método `later` acepta una instancia de `DateTime` que indica cuándo se debe enviar el mensaje:

    $when = Carbon\Carbon::now()->addMinutes(10);
    
    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->later($when, new OrderShipped($order));
    

#### Incluir a Colas Específicas

Como todas las clases mailable generadas utilizando el comando `make:mail` hacen uso del trait `Illuminate\Bus\Queueable`, se pueden llamar los métodos `onQueue` y `onConnection` en cualquier instancia de clase mailable, lo que permite especificar la conexión y el nombre de la cola para el mensaje:

    $message = (new OrderShipped($order))
                    ->onConnection('sqs')
                    ->onQueue('emails');
    
    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue($message);
    

#### Queueing por defecto

Si tiene clases mailable que desea que estén siempre en cola, se puede implementar el contrato `ShouldQueue` en la clase. Ahora, incluso si llama al método `send` cuando se envía, el mailable seguirá en cola ya que implementa el contrato:

    use Illuminate\Contracts\Queue\ShouldQueue;
    
    class OrderShipped extends Mailable implements ShouldQueue
    {
        //
    }
    

<a name="mail-and-local-development"></a>

## Correo & Desarrollo Local

Al desarrollar una aplicación que envía e-mails, es probable que no se desee enviar mensajes a direcciones de correo reales. Laravel ofrece varias maneras de "desactivar" el envío de mensajes de correo electrónico.

#### Log Driver

En lugar de enviar los correos electrónicos, el driver de correo `log` escribirá todos los mensajes de correo electrónico en los logs para su inspección. Para más información de como configurar una aplicación para el desarrollo, consultar el apartado de [documentación de configuración](/docs/{{version}}/configuration#environment-configuration).

#### Recipiente Universal

Otra solución que proporciona Laravel es crear un receptor universal de todos los correos electrónicos enviados por el framework. De esta manera, todos los correos electrónicos generados por la aplicación serán enviados a una dirección específica, en lugar de la dirección especificada cuando se envía el mensaje. Esto se puede hacer a través de la opción `to` del archivo de configuración `config/mail.php`:

    'to' => [
        'address' => 'example@example.com',
        'name' => 'Example'
    ],
    

#### Mailtrap

Por último, es posible utilizar un servicio como [Mailtrap](https://mailtrap.io) y el driver `smtp` para enviar mensajes de correo electrónico a un buzón "ficticio" el cual se puede ver como un verdadero cliente de correo. Este enfoque tiene la ventaja de que permite inspeccionar realmente los correos electrónicos finales en el visor de mensajes de Mailtrap.

<a name="events"></a>

## Eventos

Laravel dispara dos eventos durante el proceso de envío de mensajes de correo. El evento `MessageSending` se dispara antes de que se envíe un mensaje, mientras que el evento `MessageSent` se dispara después de que un mensaje ha sido enviado. Recuerde, estos eventos se disparan cuando el correo se *envía*, no cuando está en cola. Puede registrar un listener para este evento en su `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Mail\Events\MessageSending' => [
            'App\Listeners\LogSendingMessage',
        ],
        'Illuminate\Mail\Events\MessageSent' => [
            'App\Listeners\LogSentMessage',
        ],
    ];