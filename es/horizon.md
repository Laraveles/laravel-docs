# Laravel Horizon

- [Introducción](#introduction)
- [Instalación](#installation) 
    - [Configuración](#configuration)
    - [Autenticación del Dashboard](#dashboard-authentication)
- [Ejecutar Horizon](#running-horizon) 
    - [Desplegar Horizon](#deploying-horizon)
- [Etiquetas](#tags)
- [Notificaciones](#notifications)
- [Métricas](#metrics)

<a name="introduction"></a>

## Introducción

Horizon proporciona un hermoso *dashboard* y una configuración controlada por código para sus colas de Redis gestionadas por Laravel. Horizon permite monitorear de forma sencilla las métricas clave de su sistema de colas (*queues*), tales como el rendimiento de trabajos (*jobs*), el tiempo de ejecución y las fallas del trabajos.

Toda la configuración se almacena en un único y sencillo archivo de configuración, permitiendo que permanezca en el lugar de control del código donde todo su equipo puede colaborar.

<a name="installation"></a>

## Instalación

> {note} Debido a su uso de señales de proceso asincrónicas, Horizon requiere PHP 7.1+.

Puede usar Composer para instalar su proyecto Laravel:

    composer require laravel/horizon
    

Después de instalar Horizon, publique sus recursos (*assets*) usando el comando Artisan `vendor:publish`:

    php artisan vendor:publish --provider="Laravel\Horizon\HorizonServiceProvider"
    

<a name="configuration"></a>

### Configuración

Después de publicar los *assets* de Horizon, su fichero de configuración principal se ubicará en `config/horizon.php`. Este fichero de configuración permite configurar las opciones de sus *workers* y cada opción incluye una descripción de su propósito, por lo tanto, asegúrese de explorar este fichero completamente.

#### Opciones de balance

Horizon le permite elegir entre tres estrategias de balance: `simple`, `auto`, y `false`. La estrategia `simple`, la cual es la predeterminada, divide los trabajos entrantes uniformemente entre los procesos:

    'balance' => 'simple',
    

La estrategia `auto` ajusta el número de procesos de trabajo por cola basada en la actual carga de trabajo de la cola. Por ejemplo, si su cola de `notifications` tiene 1,000 trabajos en espera mientras su cola `render` está vacía, Horizon distribuirá más trabajos a su cola de `notifications` hasta que esté vacía. Cuando la opción `balance` se establece a `false`, se utilizará el funcionamiento predeterminado de Laravel, el cual procesa colas en el orden que se estableció en la configuración.

<a name="dashboard-authentication"></a>

### Autenticación del Dashboard

Horizon expone un *dashboard* en `/horizon`. Por defecto, solo podrá acceder a este *dashboard* en el entorno `local`. Para definir una política de acceso más específica para el *dashboard*, deberá usar el método `Horizon::auth`. El método `auth` acepta un *Callback* la cual deberá retornar `true` o `false`, indicando si el usuario debería tener acceso al *dashboard* de Horizon:

    Horizon::auth(function ($request) {
        // return true / false;
    });
    

<a name="running-horizon"></a>

## Ejecutar Horizon

Una vez que haya configurado sus *workers* en el fichero de configuración `config/horizon.php`, puede iniciar Horizon usando el comando Artisan `horizon`. Este único comando iniciará todos los *workers* configurados:

    php artisan horizon
    

Puede pausar el proceso de Horizon e instruirlo para continuar procesando trabajos utilizando `horizon:pause` y los comandos Artisan `horizon:continue`:

    php artisan horizon:pause
    
    php artisan horizon:continue
    

Puede finalizar el proceso principal de Horizon utilizando el comando Artisan `horizon:terminate`. Todos los trabajos que Horizon esté ejecutando en ese momento se completarán y a continuación se detendrá su ejecución:

    php artisan horizon:terminate
    

<a name="deploying-horizon"></a>

### Desplegar Horizon

If you are deploying Horizon to a live server, you should configure a process monitor to monitor the `php artisan horizon` command and restart it if it quits unexpectedly. When deploying fresh code to your server, you will need to instruct the master Horizon process to terminate so it can be restarted by your process monitor and receive your code changes.

You may gracefully terminate the master Horizon process on your machine using the `horizon:terminate` Artisan command. Todos los trabajos que Horizon esté ejecutando en ese momento se completarán y a continuación se detendrá su ejecución:

    php artisan horizon:terminate
    

#### Configuración de Supervisor

Si está utilizando el monitor de procesos *Supervisor* para gestionar su proceso `horizon`, el siguiente fichero de configuración deberá ser suficiente:

    [program:horizon]
    process_name=%(program_name)s
    command=php /home/forge/app.com/artisan horizon
    autostart=true
    autorestart=true
    user=forge
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/horizon.log
    

> {tip} Si no se encuentra cómodo gestionando sus propios servidores, considere utilizar [Laravel Forge](https://forge.laravel.com). Forge provee servidores PHP 7+ con todo lo que necesita para ejecutar aplicaciones modernas y robustas de Laravel con Horizon.

<a name="tags"></a>

## Etiquetas

Horizon permite asignar "etiquetas" a trabajos, incluyendo *mailables*, *event broadcasting*, notificaciones y colas de *event listeners*. De hecho, Horizon etiquetará de forma inteligente y automática la mayoría de los trabajos dependiendo de los modelos Eloquent que estén relacionados con el trabajo. Por ejemplo, revise el siguiente *job*:

    <?php
    
    namespace App\Jobs;
    
    use App\Video;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;
    
    class RenderVideo implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
    
        /**
         * The video instance.
         *
         * @var \App\Video
         */
        public $video;
    
        /**
         * Create a new job instance.
         *
         * @param  \App\Video  $video
         * @return void
         */
        public function __construct(Video $video)
        {
            $this->video = $video;
        }
    
        /**
         * Execute the job.
         *
         * @return void
         */
        public function handle()
        {
            //
        }
    }
    

Si este trabajo se encuentra en una cola con una instancia `App\Video`, por ejemplo, que tiene un `id` con valor `1`, automáticamente recibirá la etiqueta `App\Video:1`. Esto se debe a que Horizon examina las propiedades del trabajo buscando modelos Eloquent. Si se encuentran modelos Eloquent, Horizon inteligentemente etiquetará el trabajo utilizando el nombre de la clase y la clave principal del modelo:

    $video = App\Video::find(1);
    
    App\Jobs\RenderVideo::dispatch($video);
    

#### Etiquetado manual

If you would like to manually define the tags for one of your queueable objects, you may define a `tags` method on the class:

    class RenderVideo implements ShouldQueue
    {
        /**
         * Get the tags that should be assigned to the job.
         *
         * @return array
         */
        public function tags()
        {
            return ['render', 'video:'.$this->video->id];
        }
    }
    

<a name="notifications"></a>

## Notifications

> **Note:** Before using notifications, you should add the `guzzlehttp/guzzle` Composer package to your project. When configuring Horizon to send SMS notifications, you should also review the [prerequisites for the Nexmo notification driver](https://laravel.com/docs/5.5/notifications#sms-notifications).

If you would like to be notified when one of your queues has a long wait time, you may use the `Horizon::routeMailNotificationsTo`, `Horizon::routeSlackNotificationsTo`, and `Horizon::routeSmsNotificationsTo` methods. You may call these methods from your application's `AppServiceProvider`:

    Horizon::routeMailNotificationsTo('example@example.com');
    Horizon::routeSlackNotificationsTo('slack-webhook-url', '#channel');
    Horizon::routeSmsNotificationsTo('15556667777');
    

#### Configuring Notification Wait Time Thresholds

You may configure how many seconds are considered a "long wait" within your `config/horizon.php` configuration file. The `waits` configuration option within this file allows you to control the long wait threshold for each connection / queue combination:

    'waits' => [
        'redis:default' => 60,
    ],
    

<a name="metrics"></a>

## Metrics

Horizon includes a metrics dashboard which provides information on your job and queue wait times and throughput. In order to populate this dashboard, you should configure Horizon's `snapshot` Artisan command to run every five minutes via your application's [scheduler](/docs/{{version}}/scheduling):

    /**
     * Define the application's command schedule.
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->command('horizon:snapshot')->everyFiveMinutes();
    }