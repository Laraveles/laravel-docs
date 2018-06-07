# Colas – Queues

- [Introducción](#introduction) 
    - [Conexiones y Colas](#connections-vs-queues)
    - [Prerrequisitos del driver](#driver-prerequisites)
- [Creando los trabajos(Jobs)](#creating-jobs) 
    - [Generando las Clases Trabajo](#generating-job-classes)
    - [Estructura de la Clase](#class-structure)
- [Enviar trabajos a la cola](#dispatching-jobs) 
    - [Retrasar el envió](#delayed-dispatching)
    - [Encadenado de trabajos](#job-chaining)
    - [Customizing The Queue & Connection](#customizing-the-queue-and-connection)
    - [Especificar mayor cantidad de intentos de trabajo / tiempo de espera](#max-job-attempts-and-timeout)
    - [Límites de velocidad](#rate-limiting)
    - [Manejo de Errores](#error-handling)
- [Ejecutar el Procesador de cola](#running-the-queue-worker) 
    - [Priorización de colas](#queue-priorities)
    - [Queue Workers & Deployment](#queue-workers-and-deployment)
    - [Job Expirations & Timeouts](#job-expirations-and-timeouts)
- [Supervisor Configuration](#supervisor-configuration)
- [Gestionar trabajos fallidos](#dealing-with-failed-jobs) 
    - [Cleaning Up After Failed Jobs](#cleaning-up-after-failed-jobs)
    - [Eventos de trabajos fallidos](#failed-job-events)
    - [Reintentar trabajos fallidos](#retrying-failed-jobs)
- [Eventos de tabajos](#job-events)

<a name="introduction"></a>

## Introducción

> {tip} Laravel ahora ofrece Horizon, un hermoso *dashboard* y sistema de configuración para sus colas de Redis. Consulte la documentación completa de [Horizon](/docs/{{version}}/horizon) para obtener más información.

Las colas de Laravel proporcionan una API unificada en una variedad de backends de cola diferentes, como Beanstalk, Amazon SQS, Redis o incluso una base de datos relacional. *Queues* permite posponer el procesamiento de una tarea que consume mucho tiempo, como enviar un correo electrónico, hasta un momento posterior. Aplazar estas tareas que consumen mucho tiempo acelera drásticamente las peticiones web de la aplicación.

El archivo de configuración de *queue* de trabajo se almacena en `config.queue.php`. En este archivo se encuentran las configuraciones de conexión para cada uno de los *queue drivers* incorporados con el framework, que incluye una base de datos, [Beanstalkd](https://kr.github.io/beanstalkd/), [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](https://redis.io), y un driver síncrono que ejecutará trabajos inmediatamente (para uso local). Un *queue driver* `null` también está incluido, el cual simplemente descarta los trabajos.

<a name="connections-vs-queues"></a>

### Conexiones y Colas

Antes de comenzar con las colas en Laravel, es importante comprender la distinción entre "conexiones" (*connections*) y "colas" (*queues*). En el archivo de configuración `config/queue.php`, hay una opción de configuración `connections`. Esta opción define una conexión particular a un servicio *backend* como Amazon SQS, Beanstalk o Redis. Sin embargo, cualquier *queue connection* puede tener múltiples "*queue*" que pueden considerarse como diferentes pilas o montones de trabajos en cola.

Tenga en cuenta que cada ejemplo de configuración de conexión en el archivo de configuración `queue` contiene un atributo `queue`. Esta es la *queue* predeterminada a la que se le van a mandar los trabajos cuando se envíen a una conexión dada. En otras palabras, si envía un trabajo sin definir explícitamente a qué *queue* debe enviarse, el trabajo se colocará en la *queue* definida en el atributo `queue` de la configuración de conexión:

    // This job is sent to the default queue...
    Job::dispatch();
    
    // This job is sent to the "emails" queue...
    Job::dispatch()->onQueue('emails');
    

Es posible que algunas aplicaciones no necesiten insertar trabajos en varias *queues*, sino que prefieran tener una simple *queue*. Sin embargo, enviar trabajos a varias colas puede ser especialmente útil para las aplicaciones que desean priorizar o segmentar cómo se procesan los trabajos, ya que el *queue worker*(gestor de trabajos de cola) de Laravel le permite especificar cual cola debe procesar según su prioridad. Por ejemplo, si envía trabajos a una cola `high`(prioridad alta), se ejecutara un *worker* que les dé una mayor prioridad de procesamiento:

    php artisan queue:work --queue=high,default
    

<a name="driver-prerequisites"></a>

### Pre requisitos del driver

#### Base de datos

Para usar el *queue driver* `database`, se necesita una base de datos para guardar los trabajos. Para generar una migración que cree la tabla, es necesario ejecutar el comando de Artisan `queue:table`. Una vez creada la migración, ya puede migrar la base de datos usando el comando `migrate`:

    php artisan queue:table
    
    php artisan migrate
    

#### Redis

Para usar el *queue driver* de `redis`, debe configurar la conexión a la base de datos de Redis en el fichero de configuración `config/database.php`.

Si la *queue connection* de Redis usa *Redis Cluster*, el nombre de la cola debe contener [key hash tag](https://redis.io/topics/cluster-spec#keys-hash-tags). Esto es necesario para garantizar que todas las *Redis keys* para una cola determinada se coloquen en el mismo *hash slot*:

    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => '{default}',
        'retry_after' => 90,
    ],
    

#### Otros Pre requisitos

Las siguientes son dependencias necesarias para los *drivers* mencionados:

<div class="content-list">
  <ul>
    <li>
      Amazon SQS: <code>aws/aws-sdk-php ~3.0</code>
    </li>
    <li>
      Beanstalkd: <code>pda/pheanstalk ~3.0</code>
    </li>
    <li>
      Redis: <code>predis/predis ~1.0</code>
    </li>
  </ul>
</div>

<a name="creating-jobs"></a>

## Creando los trabajos(Jobs)

<a name="generating-job-classes"></a>

### Generando las Clases Trabajo

De forma predeterminada, todos los trabajos encolables en su aplicación se almacenan en el directorio `app/Jobs`. Si el directorio `app/Jobs` no existe, se creará cuando se ejecute el comando de *Artisan* `make:job`. Puede generar un nuevo trabajo en cola utilizando la CLI de *Artisan*:

    php artisan make:job ProcessPodcast
    

La clase generada implementara la interfaz `Illuminate\Contracts\Queue\ShouldQueue`, lo que indicará a Laravel que el trabajo debe colocarse en la cola para ejecutarse de forma asíncrona.

<a name="class-structure"></a>

### Estructura de la Clase

Las *job classes* son muy simples, normalmente contienen el método `handle`, el cual es llamado cuando el trabajo es procesado por la cola. Para comenzar, echemos un vistazo a una clase de trabajo (*job class*) de ejemplo. En este ejemplo, pretendemos que administramos un servicio de publicación de podcasts y necesitamos procesar los archivos de podcasts subidos antes de que se publiquen:</p> 

    <?php
    
    namespace App\Jobs;
    
    use App\Podcast;
    use App\AudioProcessor;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;
    
    class ProcessPodcast implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
    
        protected $podcast;
    
        /**
         * Create a new job instance.
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }
    
        /**
         * Execute the job.
         *
         * @param  AudioProcessor  $processor
         * @return void
         */
        public function handle(AudioProcessor $processor)
        {
            // Process uploaded podcast...
        }
    }
    

En este ejemplo se nota que es posible enviar un [Eloquent model](/docs/{{version}}/eloquent) directamente al constructor del trabajo. Debido al *trait* `SerializesModels` que la *job class* usa, los modelos Eloquent se serializan y deserializan elegantemente cuando el trabajo se está procesando. Si el trabajo en la cola acepta un modelo Eloquent en su constructor, únicamente el identificador del modelo sera serializado en la cola. Cuando el trabajo se ejecute, el sistema de cola buscará automáticamente en la base de datos la instancia completa del modelo. Todo es totalmente transparente para la aplicación y previene inconvenientes que pueden producirse al serializar instancias completas de modelos.

El método `handle` se llama cuando la cola procesa el trabajo. Es de tener en cuenta que es posible determinar el tipo de las dependencias en el método `handle` del trabajo. El [service container](/docs/{{version}}/container) de Laravel automáticamente inyecta estas dependencias.

> {note} Los datos binarios, como los contenidos de imágenes sin formato, se deben pasar a través de la función `base64_encode` antes de pasarlos a un trabajo en cola. De lo contrario, es posible que el trabajo no se serialice correctamente en formato JSON cuando se coloca en la cola.

<a name="dispatching-jobs"></a>

## Enviar trabajos a la cola

Una vez que haya escrito su *job class*, puede enviarla utilizando el método `dispatch` de la misma clase. Los argumentos pasados al método `dispatch` se le darán al constructor de la clase *job*:

    <?php
    
    namespace App\Http\Controllers;
    
    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;
    
    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...
    
            ProcessPodcast::dispatch($podcast);
        }
    }
    

<a name="delayed-dispatching"></a>

### Retrasar el envió

Si desea retrasar la ejecución de un trabajo en cola, puede usar el método `delay` al enviar un trabajo. Por ejemplo, especifiquemos que un trabajo no debería estar disponible para su procesamiento hasta 10 minutos después de su envío:

    <?php
    
    namespace App\Http\Controllers;
    
    use Carbon\Carbon;
    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;
    
    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...
    
            ProcessPodcast::dispatch($podcast)
                    ->delay(Carbon::now()->addMinutes(10));
        }
    }
    

> {note} El servicio de cola de Amazon SQS permite un retraso máximo de 15 minutos.

<a name="job-chaining"></a>

### Encadenado de trabajos

La cadena de trabajos permite especificar una lista de los trabajos en cola que deben ejecutarse en secuencia. Si falla un trabajo de la secuencia, el resto de los trabajos no se ejecutarán. Para ejecutar una cadena de trabajos en cola, puede usar el método `withChain` en cualquiera de sus trabajos distribuibles:

    ProcessPodcast::withChain([
        new OptimizePodcast,
        new ReleasePodcast
    ])->dispatch();
    

<a name="customizing-the-queue-and-connection"></a>

### Customizing The Queue & Connection

#### Enviar trabajo a una cola determinada

Al enviar trabajos a diferentes colas, puede "categorizar" sus trabajos de cola e incluso priorizar la cantidad de procesadores que asigna a varias colas. Tenga en cuenta que esto no envía los trabajos a diferentes "conexiones" de cola definidas por su archivo de configuración de cola, sino solo a colas específicas dentro de una única conexión. Para especificar la cola, se usa el método `onQueue` al enviar el trabajo:

    <?php
    
    namespace App\Http\Controllers;
    
    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;
    
    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...
    
            ProcessPodcast::dispatch($podcast)->onQueue('processing');
        }
    }
    

#### Enviar trabajo a una conexión determinada

Si está trabajando con múltiples conexiones de colas, puede especificar a cual conexión envía el trabajo. Para especificar la conexión, use el método `onConnection` cuando envié el trabajo:

    <?php
    
    namespace App\Http\Controllers;
    
    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;
    
    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...
    
            ProcessPodcast::dispatch($podcast)->onConnection('sqs');
        }
    }
    

Puede encadenar los métodos `onConnection` y `onQueue` para especificar la conexión y la cola para el trabajo:

    ProcessPodcast::dispatch($podcast)
                  ->onConnection('sqs')
                  ->onQueue('processing');
    

<a name="max-job-attempts-and-timeout"></a>

### Especificar mayor cantidad de intentos de trabajo / tiempo de espera

#### Mayor cantidad de intentos

Un enfoque para especificar el número máximo de veces que se puede intentar un trabajo es a través del interruptor `--tries` en la línea de comando de Artisan:

    php artisan queue:work --tries=3
    

Sin embargo, puede tomar un enfoque más especifico al definir la cantidad máxima de intentos en la *job class*. Si se especifica la cantidad máxima de intentos en *job class*, tendrá prioridad sobre el valor proporcionado en la línea de comando:

    <?php
    
    namespace App\Jobs;
    
    class ProcessPodcast implements ShouldQueue
    {
        /**
         * The number of times the job may be attempted.
         *
         * @var int
         */
        public $tries = 5;
    }
    

<a name="time-based-attempts"></a>

#### Intentos basados en tiempo

Como alternativa para definir cuántos intentos debe hacer un trabajo antes de que falle, puede definir un tiempo en la que el trabajo debe expirar. Esto permite que un trabajo se intente varias veces en un tiempo determinado. Para definir la hora a la que un trabajo debe expirar, agregue un método `retryUntil` a su clase de trabajo:

    /**
     * Determine the time at which the job should timeout.
     *
     * @return \DateTime
     */
    public function retryUntil()
    {
        return now()->addSeconds(5);
    }
    

> {tip} También puede definir un método `retryUntil` en sus *queued event listeners*.

#### Tiempo limite

> {note} La característica `timeout` está optimizada para PHP 7.1+ y la extensión de PHP `pcntl`.

Del mismo modo, la cantidad máxima de segundos que se pueden ejecutar trabajos se puede especificar utilizando `--timeout` en la línea de comandos de Artisan:

    php artisan queue:work --timeout=30
    

Sin embargo, también puede definir en la *job class* la cantidad máxima de segundos que se debe permitir que un trabajo se ejecute. Si el tiempo de espera se especifica en la *job class*, tendrá prioridad sobre cualquier tiempo de espera especificado en la línea de comando:

    <?php
    
    namespace App\Jobs;
    
    class ProcessPodcast implements ShouldQueue
    {
        /**
         * The number of seconds the job can run before timing out.
         *
         * @var int
         */
        public $timeout = 120;
    }
    

<a name="rate-limiting"></a>

### Límites de velocidad

> {note} Esta función requiere que su aplicación pueda interactuar con un [Redis server](/docs/{{version}}/redis).

Si su aplicación interactúa con Redis, pondría limitar sus trabajos en cola por tiempo o concurrencia. Esta función puede ser útil cuando tus trabajos en cola interactúen con API que también tienen un límite de velocidad. Por ejemplo, con el método `throttle`, puede limitar un tipo de trabajo determinado para que se ejecute solo 10 veces cada 60 segundos. Si no se puede obtener un bloqueo, normalmente debe volver a liberar el trabajo en la cola para que pueda volver a intentarlo más adelante:

    Redis::throttle('key')->allow(10)->every(60)->then(function () {
        // Job logic...
    }, function () {
        // Could not obtain lock...
    
        return $this->release(10);
    });
    

> {tip} En el ejemplo anterior, `key` puede ser cualquier cadena que identifique de forma única el tipo de trabajo al que le gustaría calificar el límite. Por ejemplo, es posible que desee construir la *key* según el nombre de la clase del trabajo y los ID de los modelos Eloquent en los que opera.

Alternativamente, puede especificar la cantidad máxima de procesadores que pueden tratar simultáneamente un trabajo determinado. Esto puede ser útil cuando un trabajo en cola está modificando un recurso que solo debería ser modificado por un trabajo a la vez. Por ejemplo, con el método `funnel`, puede limitar trabajos de un tipo determinado para que solo los trate un procesador a la vez:

    Redis::funnel('key')->limit(1)->then(function () {
        // Job logic...
    }, function () {
        // Could not obtain lock...
    
        return $this->release(10);
    });
    

> {tip} Al usar la limitación de velocidad, es difícil determinar la cantidad de intentos que su trabajo necesitará para ejecutar con éxito. Por lo tanto, es útil combinar la limitación de velocidad con [time based attempts](#time-based-attempts).

<a name="error-handling"></a>

### Manejo de Errores

Si se lanza una excepción mientras se está procesando el trabajo, el trabajo se volverá a liberar automáticamente en la cola para que se pueda intentar nuevamente. El trabajo continuará siendo liberado hasta que se haya intentado la cantidad máxima de veces permitida por su aplicación. El número máximo de intentos se define mediante `--tries` utilizado en el comando `queue:work` de Artisan. Alternativamente, la cantidad máxima de intentos puede definirse en la *job class*. Más información sobre cómo ejecutar el procesador de cola [en siguientes secciones](#running-the-queue-worker).

<a name="running-the-queue-worker"></a>

## Ejecutar el Procesador de cola

Laravel incluye un procesador de cola que tratara los nuevos trabajos a medida que se colocan en la cola. Puede ejecutar el procesador utilizando el comando de Artisan `queue: work`. Tenga en cuenta que una vez que el comando `queue: work` haya comenzado, continuará ejecutándose hasta que se detenga manualmente o cierre su terminal:

    php artisan queue:work
    

> {tip} Para mantener el proceso `queue: work` ejecutándose permanentemente en segundo plano, debe usar un supervisor de procesos como [Supervisor](#supervisor-configuration) para asegurarse de que el procesador de cola no deje de ejecutarse.

Recuerde, los procesadores de cola son procesos de larga duración y almacenan el estado de inicio de la aplicación en memoria. Como resultado, no notarán cambios en su código una vez que se hayan iniciado. Por lo tanto, durante el proceso de implementación, asegúrese de [restart your queue workers](#queue-workers-and-deployment).

#### Processing A Single Job

La opción `--once` se puede usar para indicar al procesador de cola que solo procese un único trabajo de la cola:

    php artisan queue:work --once
    

#### Specifying The Connection & Queue

También puede especificar qué conexión de cola debe utilizar el procesadore de cola. El nombre de conexión pasado al comando `work` debe corresponder a una de las conexiones definidas en su archivo de configuración `config/queue.php`:

    php artisan queue:work redis
    

Puede personalizar aún más su procesadores de cola para que trate colas especificas para una conexión determinada. Por ejemplo, si todos sus correos electrónicos se procesan en la cola `emails` de la conexión de cola `redis`, puede ejecutar el siguiente comando para iniciar un procesador que solo trate esa cola:

    php artisan queue:work redis --queue=emails
    

#### Resource Considerations

El *Daemon* de los procesadores de cola, no reinicia el *framewoirk* antes de procesar cada trabajo. Por lo tanto, debe liberar todos los recursos pesados después de cada trabajo. Por ejemplo, si está manipulando imágenes con la biblioteca GD, debe liberar la memoria con `imagedestroy` cuando haya terminado.

<a name="queue-priorities"></a>

### Prioridades De Colas

En ocasiones, es posible que desee priorizar cómo se procesan sus colas. Por ejemplo, en su `config/queue.php` puede establecer la cola por defecto `queue` para su conexión a `redis` con prioridad `low` (baja). Sin embargo, de vez en cuando es posible que desee insertar un trabajo en una cola de prioridad `high` (alta) como la siguiente:

    dispatch((new Job)->onQueue('high'));
    

Para iniciar un procesador de cola que verifique que todas las tareas de cola de prioridad `high` se procesen antes de continuar con cualquier trabajo en la cola `low`, pase una lista delimitada por comas de nombres de cola al comando `work`:

    php artisan queue:work --queue=high,low
    

<a name="queue-workers-and-deployment"></a>

### Queue Workers & Deployment

Como los procesadores de cola son procesos de larga duración, no recibirán cambios en su código sin reiniciarse. Por lo tanto, la forma más sencilla de implementar una aplicación utilizando los procesadores de cola es reiniciar los procesadores durante el proceso de implementación. Puede reiniciar fácilmente todos los procesadores usando el comando `queue:restart`:

    php artisan queue:restart
    

Este comando indicará a todos los procesadores de cola que "mueran" después de que terminen de procesar su trabajo actual para que no se pierdan trabajos existentes. Como los procesadores de cola finalizaran cuando se ejecuta el comando `queue:restart`, debe ejecutar un administrador de procesos como [Supervisor](#supervisor-configuration) para reiniciar automáticamente los procesadores de cola.

> {tip} La cola usa la [caché](/docs/{{version}}/cache) para almacenar las señales de reinicio, así que antes de utilizar esta característica, debería verificar que el *driver* de la caché está correctamente configurado para tu aplicación.

<a name="job-expirations-and-timeouts"></a>

### Job Expirations & Timeouts

#### Job Expiration

En su archivo de configuración `config/queue.php`, cada conexión de cola define una opción `retry_after`. Esta opción especifica cuántos segundos debe esperar la conexión de cola antes de reintentar un trabajo que se está procesando. Por ejemplo, si el valor de `retry_after` se establece en `90`, el trabajo se devolverá a la cola si se ha procesado durante 90 segundos sin haber sido eliminado. Por lo general, debe establecer el valor `retry_after` en la cantidad máxima de segundos que sus trabajos deben razonablemente tomar para completar el procesamiento.

> {note} La única conexión de cola que no contiene un valor `retry_after` es Amazon SQS. SQS volverá a intentar el trabajo según el [Default Visibility Timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html) que se administra en la consola de AWS.

#### Worker Timeouts

El comando de Artisan `queue:work` expone una opción `--timeout`. La opción `--timeout` especifica cuánto esperará el proceso maestro de cola de Laravel antes de matar un procesador de cola hijo que está tratando un trabajo. En ocasiones, un proceso de cola hijo puede "congelarse" por varias razones, como una llamada HTTP externa que no responde. La opción `--timeout` elimina los procesos inmovilizados que han excedido el límite de tiempo especificado:

    php artisan queue:work --timeout=60
    

La opción de configuración `retry_after` y la opción CLI `--timeout` son diferentes, pero funcionan juntas para garantizar que no se pierdan trabajos y que los trabajos solo se procesen satisfactoriamente una vez.

> {note} El valor `--timeout` siempre debe ser al menos varios segundos más corto que su valor de configuración `retry_after`. Esto asegurará que un procesador de cola que procesa un trabajo determinado siempre muera antes de volver a intentarlo. Si su opción `--timeout` es mayor que el valor de configuración `retry_after`, sus trabajos pueden procesarse dos veces.

#### Worker Sleep Duration

Cuando hay trabajos disponibles en la cola, el procesador de cola seguirá procesando trabajos sin demora entre ellos. Sin embargo, la opción `sleep` determina cuánto tiempo "dormirá" el procesador si no hay nuevos trabajos disponibles. Mientras duerme, el procesador de cola no procesará ningún trabajo nuevo; los trabajos se procesarán después de que el procesador se despierte nuevamente.

    php artisan queue:work --sleep=3
    

<a name="supervisor-configuration"></a>

## Configuración de Supervisor

#### Installing Supervisor

Supervisor es un monitor de procesos para el sistema operativo Linux y reiniciará automáticamente su proceso `queue:work` si falla. Para instalar Supervisor en Ubuntu, se puede usar el siguiente comando:

    sudo apt-get install supervisor
    

> {tip} Si configurar el supervisor por ud mismo suena abrumador, considere usar [Laravel Forge](https://forge.laravel.com), que instalará y configurará automáticamente el supervisor para sus proyectos de Laravel.

#### Configuring Supervisor

Los archivos de configuración de Supervisor típicamente se almacenan en el directorio `/etc/supervisor/conf.d`. Dentro de este directorio, se pueden crear cualquier número de archivos de configuración que le indiquen a Supervisor cómo deberían ser monitoreados los procesos. Por ejemplo, creemos el archivo `laravel-worker.conf` que inicia y monitorea un proceso `queue:work`:

    [program:laravel-worker]
    process_name=%(program_name)s_%(process_num)02d
    command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3
    autostart=true
    autorestart=true
    user=forge
    numprocs=8
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/worker.log
    

En este ejemplo, la directiva `numprocs` instruye a Supervisor para que ejecute 8 procesos `queue:work` y los monitoree a todos, reiniciándolos si alguno falla. Por supuesto, debe cambiar la sección `queue:work sqs` de la instrucción `command` para reflejar su conexión de cola deseada.

#### Starting Supervisor

Una vez que se ha creado el archivo de configuración, se puede actualizar la configuración de Supervisor e iniciar los procesos usando los siguientes comandos:

    sudo supervisorctl reread
    
    sudo supervisorctl update
    
    sudo supervisorctl start laravel-worker:*
    

Para obtener más información sobre Supervisor, consulte la [Supervisor documentation](http://supervisord.org/index.html).

<a name="dealing-with-failed-jobs"></a>

## Gestionar Trabajos Fallidos

Algunas veces sus trabajos en cola fallarán. ¡No te preocupes, las cosas no siempre salen según lo planeado! Laravel incluye una forma conveniente de especificar la cantidad máxima de intentos de trabajo. Después de que un trabajo haya excedido esta cantidad de intentos, se insertará en la tabla de la base de datos `failed_jobs`. Para crear una migración para la tabla `failed_jobs`, puede usar el comando `queue:failed-table`:

    php artisan queue:failed-table
    
    php artisan migrate
    

Luego, al ejecutar el [queue worker](#running-the-queue-worker), debe especificar la cantidad máxima de veces que se debe intentar un trabajo utilizando `--tries` en el comando `queue:work`. Si no especifica un valor para la opción `--tries`, los trabajos se intentarán indefinidamente:

    php artisan queue:work redis --tries=3
    

<a name="cleaning-up-after-failed-jobs"></a>

### Cleaning Up After Failed Jobs

Puede definir un método `failed` directamente en su clase de trabajo, lo que le permite realizar tareas de limpieza específicas cuando ocurre una falla. Esta es la ubicación perfecta para enviar una alerta a sus usuarios o revertir cualquier acción realizada por el trabajo. La `Exception` que provocó la falla del trabajo se pasará al método `failed`:

    <?php
    
    namespace App\Jobs;
    
    use Exception;
    use App\Podcast;
    use App\AudioProcessor;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    
    class ProcessPodcast implements ShouldQueue
    {
        use InteractsWithQueue, Queueable, SerializesModels;
    
        protected $podcast;
    
        /**
         * Create a new job instance.
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }
    
        /**
         * Execute the job.
         *
         * @param  AudioProcessor  $processor
         * @return void
         */
        public function handle(AudioProcessor $processor)
        {
            // Process uploaded podcast...
        }
    
        /**
         * The job failed to process.
         *
         * @param  Exception  $exception
         * @return void
         */
        public function failed(Exception $exception)
        {
            // Send user notification of failure, etc...
        }
    }
    

<a name="failed-job-events"></a>

### Eventos de Trabajos Fallidos

Si desea registrar un evento que será invocado cuando falla un trabajo, puede usar el método `Queue::failing`. Este evento es una gran oportunidad para notificar a su equipo por correo electrónico o por [HipChat](https://www.hipchat.com). Por ejemplo, es posible adjuntar un *callback* a este evento desde el `AppServiceProvider` incluido con Laravel:

    <?php
    
    namespace App\Providers;
    
    use Illuminate\Support\Facades\Queue;
    use Illuminate\Queue\Events\JobFailed;
    use Illuminate\Support\ServiceProvider;
    
    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Queue::failing(function (JobFailed $event) {
                // $event->connectionName
                // $event->job
                // $event->exception
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
    

<a name="retrying-failed-jobs"></a>

### Reintentar Trabajos Fallidos

Para ver todos sus trabajos fallidos que se han insertado en su tabla de base de datos `failed_jobs`, puede usar el comando de Artisan `queue:failed`:

    php artisan queue:failed
    

El comando `queue:failed` mostrará la identificación del trabajo, la conexión, la cola y el tiempo de falla. El ID del trabajo se puede usar para volver a intentar el trabajo fallido. Por ejemplo, para volver a intentar un trabajo fallido que tiene una ID de `5`, ejecute el siguiente comando:

    php artisan queue:retry 5
    

Para volver a intentar todos sus trabajos fallidos, ejecute el comando `queue:retry` y pase `all` como ID:

    php artisan queue:retry all
    

Si desea eliminar un trabajo fallido, puede usar el comando `queue:forget`:

    php artisan queue:forget 5
    

Para eliminar todos sus trabajos fallidos, puede usar el comando `queue:flush`:

    php artisan queue:flush
    

<a name="job-events"></a>

## Eventos de Tabajos

Utilizando los métodos `before` y `after` en la `Queue` [facade](/docs/{{version}}/facades), puede especificar los *callbacks * para que se ejecuten antes o después de una trabajo en cola es procesado. Estos *callbacks* son una gran oportunidad para realizar estadísticas de incremento o registro adicionales para un *dashboard*. Por lo general, debe llamar a estos métodos desde un [service provider](/docs/{{version}}/providers). Por ejemplo, podemos usar el `AppServiceProvider` que se incluye con Laravel:

    <?php
    
    namespace App\Providers;
    
    use Illuminate\Support\Facades\Queue;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Queue\Events\JobProcessed;
    use Illuminate\Queue\Events\JobProcessing;
    
    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Queue::before(function (JobProcessing $event) {
                // $event->connectionName
                // $event->job
                // $event->job->payload()
            });
    
            Queue::after(function (JobProcessed $event) {
                // $event->connectionName
                // $event->job
                // $event->job->payload()
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
    

Al utilizar el método `looping` en la `Queue` facade</1, puede especificar los *callbacks* que se ejecutan antes de que el procesador de cola intente recuperar un trabajo de una cola. Por ejemplo, puede registrar un *Closure* para deshacer cualquier transacción que haya quedado abierta por un trabajo que falló:</p> 

    Queue::looping(function () {
        while (DB::transactionLevel() > 0) {
            DB::rollBack();
        }
    });