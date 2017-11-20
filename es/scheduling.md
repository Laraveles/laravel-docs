# Programación de Tareas – *Task Scheduling*

- [Introducción](#introduction)
- [Definición de tiempos](#defining-schedules) 
    - [Programación de comandos de Artisan](#scheduling-artisan-commands)
    - [Programación de colas de trabajo](#scheduling-queued-jobs)
    - [Programación de comandos del terminal](#scheduling-shell-commands)
    - [Opciones de frecuencia de la programación](#schedule-frequency-options)
    - [Prevención de solapamientos de tareas](#preventing-task-overlaps)
    - [Modo de mantenimiento](#maintenance-mode)
- [Salida generada](#task-output)
- [*Hooks* de tareas](#task-hooks)

<a name="introduction"></a>

## Introducción

En el pasado, es posible que haya generado una entrada Cron para cada tarea que necesite programar en su servidor. Sin embargo, esto puede convertirse rápidamente en una molestia, porque su programador de tareas ya no está en su origen y debe usar SSH en su servidor para agregar entradas adicionales de Cron.

El programador de comandos de Laravel le permite definir con fluidez y expresividad su programación de comandos dentro del propio Laravel. Cuando utilice el programador, sólo se necesita una única entrada Cron en su servidor. La tarea programada se define en el método `schedule` en el archivo `app/Console/Kernel.php`. Para ayudarle a empezar, se incluye un ejemplo sencillo con el método.

### Iniciando el programador

Cuando utilice el programador, sólo necesita añadir la siguiente entrada Cron a su servidor. Si no sabe cómo agregar entradas Cron a su servidor, considere utilizar un servicio como [Laravel Forge](https://forge.laravel.com) que puede administrar las entradas Cron por usted:

    * * * * * php /path-to-your-project/artisan schedule:run >> /dev/null 2>&1
    

Este Cron llamará al programador de Laravel cada minuto. Cuando se ejecuta el comando `schedule:run`, Laravel evaluará las tareas programadas y ejecutará las que se deben ejecutar.

<a name="defining-schedules"></a>

## Definición de tiempos

Puede definir todas las tareas en el método de `schedule` de la clase `App\Console\Kernel`. Para comenzar, veamos un ejemplo de coordinar una tarea. En este ejemplo, coordinamos un `Closure` que se llamará todos los días a medianoche. Dentro del `Closure` ejecutaremos una consulta de base de datos para borrar una tabla:

    <?php
    
    namespace App\Console;
    
    use DB;
    use Illuminate\Console\Scheduling\Schedule;
    use Illuminate\Foundation\Console\Kernel as ConsoleKernel;
    
    class Kernel extends ConsoleKernel
    {
        /**
         * The Artisan commands provided by your application.
         *
         * @var array
         */
        protected $commands = [
            \App\Console\Commands\Inspire::class,
        ];
    
        /**
         * Define the application's command schedule.
         *
         * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
         * @return void
         */
        protected function schedule(Schedule $schedule)
        {
            $schedule->call(function () {
                DB::table('recent_users')->delete();
            })->daily();
        }
    }
    

<a name="scheduling-artisan-commands"></a>

### Programación de comandos de Artisan

Además de programar *Closures*, también puede programar [comandos de Artisan](/docs/{{version}}/artisan) y comandos del sistema operativo. Por ejemplo, puede usar el método `command` para programar un comando Artisan usando el nombre o la clase del comando:

    $schedule->command('emails:send --force')->daily();
    
    $schedule->command(EmailsCommand::class, ['--force'])->daily();
    

<a name="scheduling-queued-jobs"></a>

### Programación de colas de trabajo

El método `job` se puede utilizar para programar un [trabajo en cola (queued job)](/docs/{{version}}/queues). Este método proporciona una manera conveniente de programar trabajos sin usar el método `call` para crear cierres manuales al colocarlo en la cola:

    $schedule->job(new Heartbeat)->everyFiveMinutes();
    

<a name="scheduling-shell-commands"></a>

### Programación de comandos del terminal

El método `exec` puede usarse para enviar una orden al sistema operativo:

    $schedule->exec('node /home/forge/script.js')->daily();
    

<a name="schedule-frequency-options"></a>

### Opciones de frecuencia de la programación

Por supuesto, hay una variedad de horarios que usted puede asignar a su tarea:

| Método                               | Descripción                                                              |
| ------------------------------------ | ------------------------------------------------------------------------ |
| `->cron('* * * * * *');`          | Ejecutar la tarea en un cronograma personalizado de Cron                 |
| `->everyMinute();`                | Ejecutar la tarea cada minuto                                            |
| `->everyFiveMinutes();`           | Ejecutar la tarea cada cinco minutos                                     |
| `->everyTenMinutes();`            | Ejecutar la tarea cada diez minutos                                      |
| `->everyFifteenMinutes();`        | Ejecutar la tarea cada quince minutos                                    |
| `->everyThirtyMinutes();`         | Ejecutar la tarea cada treinta minutos                                   |
| `->hourly();`                     | Ejecutar la tarea cada hora                                              |
| `->hourlyAt(17);`                 | Ejecutar la tarea cada hora a los 17 minutos después de la hora en punto |
| `->daily();`                      | Ejecutar la tarea todos los días a medianoche                            |
| `->dailyAt('13:00');`             | Ejecutar la tarea todos los días a las 13:00                             |
| `->twiceDaily(1, 13);`            | Ejecutar la tarea diariamente a la 1:00 y a las 13:00                    |
| `->weekly();`                     | Ejecutar la tarea cada semana                                            |
| `->monthly();`                    | Ejecutar la tarea cada mes                                               |
| `->monthlyOn(4, '15:00');`        | Ejecutar la tarea cada mes el día 4 a las 15:00                          |
| `->quarterly();`                  | Ejecutar la tarea cada trimestre                                         |
| `->yearly();`                     | Ejecutar la tarea cada año                                               |
| `->timezone('America/New_York');` | Configurar la zona horaria                                               |

Estos métodos pueden combinarse con restricciones adicionales para crear programas aún más ajustados que sólo funcionan en ciertos días de la semana. Por ejemplo, para programar un comando que se ejecute semanalmente el lunes:

    // Run once per week on Monday at 1 PM...
    $schedule->call(function () {
        //
    })->weekly()->mondays()->at('13:00');
    
    // Run hourly from 8 AM to 5 PM on weekdays...
    $schedule->command('foo')
              ->weekdays()
              ->hourly()
              ->timezone('America/Chicago')
              ->between('8:00', '17:00');
    

A continuación se muestra una lista de las restricciones adicionales del horario:

| Método                        | Descripción                                                    |
| ----------------------------- | -------------------------------------------------------------- |
| `->weekdays();`            | Limita la tarea a días de la semana                            |
| `->sundays();`             | Limita la tarea al domingo                                     |
| `->mondays();`             | Limita la tarea al lunes                                       |
| `->tuesdays();`            | Limita la tarea al martes                                      |
| `->wednesdays();`          | Limita la tarea al miércoles                                   |
| `->thursdays();`           | Limita la tarea al jueves                                      |
| `->fridays();`             | Limita la tarea al viernes                                     |
| `->saturdays();`           | Limita la tarea al sábado                                      |
| `->between($start, $end);` | Limita la tarea a ejecutarse entre las horas de inicio y final |
| `->when(Closure);`         | Limita la tarea basada en una prueba de verdad                 |

#### Restricciones de tiempo

Se puede utilizar el método `between` para limitar la ejecución de una tarea basada en la hora del día:

    $schedule->command('reminders:send')
                        ->hourly()
                        ->between('7:00', '22:00');
    

Del mismo modo, el método `unlessBetween` puede utilizarse para excluir la ejecución de una tarea durante un período de tiempo:

    $schedule->command('reminders:send')
                        ->hourly()
                        ->unlessBetween('23:00', '4:00');
    

#### Restricciones con pruebas de verdad

El método `when` puede utilizarse para limitar la ejecución de una tarea basada en el resultado de una prueba de verdad dada. En otras palabras, si el `Closure` devuelve `true`, la tarea se ejecutará siempre y cuando ninguna otra condición de restricción impida que la tarea se ejecute:

    $schedule->command('emails:send')->daily()->when(function () {
        return true;
    });
    

El método `skip` puede verse como el inverso de `when`. Si el método `skip` devuelve `true`, la tarea programada no se ejecutará:

    $schedule->command('emails:send')->daily()->skip(function () {
        return true;
    });
    

Cuando se utilizan métodos encadenados `when`, el comando programado sólo se ejecutará si todos devuelven `true`.

<a name="preventing-task-overlaps"></a>

### Prevención de solapamientos de tareas

De forma predeterminada, las tareas programadas se ejecutarán aunque la instancia anterior de la tarea siga en ejecución. Para evitarlo, puede utilizar el método `withoutOverlapping`:

    $schedule->command('emails:send')->withoutOverlapping();
    

En este ejemplo, el [comando Artisan](/docs/{{version}}/artisan) `emails:send` se ejecutará cada minuto si no está funcionando. El método `withoutOverlapping` es especialmente útil si tiene tareas que varían drásticamente en su tiempo de ejecución, impidiéndole predecir exactamente cuánto tiempo tomará una tarea dada.

<a name="maintenance-mode"></a>

### Modo de mantenimiento

Las tareas programadas de Laravel no se ejecutarán cuando Laravel está en [modo mantenimiento](/docs/{{version}}/configuration#maintenance-mode), ya que no es deseable que las tareas interfieran con cualquier mantenimiento inacabado que pueda estar realizándose en el servidor. Sin embargo, si desea forzar una tarea para que se ejecute incluso en modo de mantenimiento, puede utilizar el método `evenInMaintenanceMode`:

    $schedule->command('emails:send')->evenInMaintenanceMode();
    

<a name="task-output"></a>

## Salida generada

El programador de Laravel proporciona varios métodos convenientes para trabajar con la salida generada por las tareas programadas. En primer lugar, utilizando el método `sendOutputTo`, puede enviar la salida a un fichero para su posterior inspección:

    $schedule->command('emails:send')
             ->daily()
             ->sendOutputTo($filePath);
    

Si desea añadir la salida a un archivo determinado, puede utilizar el método `appendOutputTo`:

    $schedule->command('emails:send')
             ->daily()
             ->appendOutputTo($filePath);
    

Utilizando el método `emailOutputTo`, puede enviar la salida a una dirección de correo electrónico de su elección. Antes de enviar por correo electrónico la salida de una tarea, debe configurar los [servicios de correo electrónico](/docs/{{version}}/mail) de Laravel:

    $schedule->command('foo')
             ->daily()
             ->sendOutputTo($filePath)
             ->emailOutputTo('foo@example.com');
    

> {note} Los métodos `emailOutputTo`, `sendOutputTo` y `appendOutputTo` son exclusivos del método `command` y no son compatibles con `call`.

<a name="task-hooks"></a>

## *Hooks* de tareas

Utilizando los métodos `before` y `after`, puede especificar el código a ejecutar antes y después de que la tarea programada haya finalizado:

    $schedule->command('emails:send')
             ->daily()
             ->before(function () {
                 // Task is about to start...
             })
             ->after(function () {
                 // Task is complete...
             });
    

#### Ping a URLs

Utilizando los métodos `pingBefore` y `thenPing`, el programador puede hacer *ping* automáticamente a una URL determinada antes o después de que se complete una tarea. Este método es útil para notificar a un servicio externo, como [Laravel Envoyer](https://envoyer.io), que su tarea programada está comenzando o ha finalizado:

    $schedule->command('emails:send')
             ->daily()
             ->pingBefore($url)
             ->thenPing($url);
    

El uso de la característica `pingBefore($url)` o `thenPing($url)` requiere la biblioteca Guzzle HTTP. Puedes añadir Guzzle a tu proyecto usando el gestor de paquetes Composer:

    composer require guzzlehttp/guzzle