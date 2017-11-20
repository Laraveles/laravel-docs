# Task Scheduling

- [Introduction](#introduction)
- [Defining Schedules](#defining-schedules) 
    - [Scheduling Artisan Commands](#scheduling-artisan-commands)
    - [Scheduling Queued Jobs](#scheduling-queued-jobs)
    - [Scheduling Shell Commands](#scheduling-shell-commands)
    - [Schedule Frequency Options](#schedule-frequency-options)
    - [Preventing Task Overlaps](#preventing-task-overlaps)
    - [Maintenance Mode](#maintenance-mode)
- [Task Output](#task-output)
- [Task Hooks](#task-hooks)

<a name="introduction"></a>

## Introduction

In the past, you may have generated a Cron entry for each task you needed to schedule on your server. However, this can quickly become a pain, because your task schedule is no longer in source control and you must SSH into your server to add additional Cron entries.

Laravel's command scheduler allows you to fluently and expressively define your command schedule within Laravel itself. When using the scheduler, only a single Cron entry is needed on your server. Your task schedule is defined in the `app/Console/Kernel.php` file's `schedule` method. To help you get started, a simple example is defined within the method.

### Starting The Scheduler

When using the scheduler, you only need to add the following Cron entry to your server. If you do not know how to add Cron entries to your server, consider using a service such as [Laravel Forge](https://forge.laravel.com) which can manage the Cron entries for you:

    * * * * * php /path-to-your-project/artisan schedule:run >> /dev/null 2>&1
    

This Cron will call the Laravel command scheduler every minute. When the `schedule:run` command is executed, Laravel will evaluate your scheduled tasks and runs the tasks that are due.

<a name="defining-schedules"></a>

## Defining Schedules

You may define all of your scheduled tasks in the `schedule` method of the `App\Console\Kernel` class. To get started, let's look at an example of scheduling a task. In this example, we will schedule a `Closure` to be called every day at midnight. Within the `Closure` we will execute a database query to clear a table:

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

### Scheduling Artisan Commands

In addition to scheduling Closure calls, you may also schedule [Artisan commands](/docs/{{version}}/artisan) and operating system commands. For example, you may use the `command` method to schedule an Artisan command using either the command's name or class:

    $schedule->command('emails:send --force')->daily();
    
    $schedule->command(EmailsCommand::class, ['--force'])->daily();
    

<a name="scheduling-queued-jobs"></a>

### Scheduling Queued Jobs

The `job` method may be used to schedule a [queued job](/docs/{{version}}/queues). This method provides a convenient way to schedule jobs without using the `call` method to manually create Closures to queue the job:

    $schedule->job(new Heartbeat)->everyFiveMinutes();
    

<a name="scheduling-shell-commands"></a>

### Scheduling Shell Commands

The `exec` method may be used to issue a command to the operating system:

    $schedule->exec('node /home/forge/script.js')->daily();
    

<a name="schedule-frequency-options"></a>

### Schedule Frequency Options

Of course, there are a variety of schedules you may assign to your task:

| Method                               | Description                                      |
| ------------------------------------ | ------------------------------------------------ |
| `->cron('* * * * * *');`          | Run the task on a custom Cron schedule           |
| `->everyMinute();`                | Run the task every minute                        |
| `->everyFiveMinutes();`           | Run the task every five minutes                  |
| `->everyTenMinutes();`            | Run the task every ten minutes                   |
| `->everyFifteenMinutes();`        | Run the task every fifteen minutes               |
| `->everyThirtyMinutes();`         | Run the task every thirty minutes                |
| `->hourly();`                     | Run the task every hour                          |
| `->hourlyAt(17);`                 | Run the task every hour at 17 mins past the hour |
| `->daily();`                      | Run the task every day at midnight               |
| `->dailyAt('13:00');`             | Run the task every day at 13:00                  |
| `->twiceDaily(1, 13);`            | Run the task daily at 1:00 & 13:00               |
| `->weekly();`                     | Run the task every week                          |
| `->monthly();`                    | Run the task every month                         |
| `->monthlyOn(4, '15:00');`        | Run the task every month on the 4th at 15:00     |
| `->quarterly();`                  | Run the task every quarter                       |
| `->yearly();`                     | Run the task every year                          |
| `->timezone('America/New_York');` | Set the timezone                                 |

These methods may be combined with additional constraints to create even more finely tuned schedules that only run on certain days of the week. For example, to schedule a command to run weekly on Monday:

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
    

Below is a list of the additional schedule constraints:

| Method                        | Description                                       |
| ----------------------------- | ------------------------------------------------- |
| `->weekdays();`            | Limit the task to weekdays                        |
| `->sundays();`             | Limit the task to Sunday                          |
| `->mondays();`             | Limit the task to Monday                          |
| `->tuesdays();`            | Limit the task to Tuesday                         |
| `->wednesdays();`          | Limit the task to Wednesday                       |
| `->thursdays();`           | Limit the task to Thursday                        |
| `->fridays();`             | Limit the task to Friday                          |
| `->saturdays();`           | Limit the task to Saturday                        |
| `->between($start, $end);` | Limit the task to run between start and end times |
| `->when(Closure);`         | Limit the task based on a truth test              |

#### Between Time Constraints

The `between` method may be used to limit the execution of a task based on the time of day:

    $schedule->command('reminders:send')
                        ->hourly()
                        ->between('7:00', '22:00');
    

Similarly, the `unlessBetween` method can be used to exclude the execution of a task for a period of time:

    $schedule->command('reminders:send')
                        ->hourly()
                        ->unlessBetween('23:00', '4:00');
    

#### Truth Test Constraints

The `when` method may be used to limit the execution of a task based on the result of a given truth test. In other words, if the given `Closure` returns `true`, the task will execute as long as no other constraining conditions prevent the task from running:

    $schedule->command('emails:send')->daily()->when(function () {
        return true;
    });
    

The `skip` method may be seen as the inverse of `when`. If the `skip` method returns `true`, the scheduled task will not be executed:

    $schedule->command('emails:send')->daily()->skip(function () {
        return true;
    });
    

When using chained `when` methods, the scheduled command will only execute if all `when` conditions return `true`.

<a name="preventing-task-overlaps"></a>

### Preventing Task Overlaps

By default, scheduled tasks will be run even if the previous instance of the task is still running. To prevent this, you may use the `withoutOverlapping` method:

    $schedule->command('emails:send')->withoutOverlapping();
    

In this example, the `emails:send` [Artisan command](/docs/{{version}}/artisan) will be run every minute if it is not already running. The `withoutOverlapping` method is especially useful if you have tasks that vary drastically in their execution time, preventing you from predicting exactly how long a given task will take.

<a name="maintenance-mode"></a>

### Maintenance Mode

Laravel's scheduled tasks will not run when Laravel is in [maintenance mode](/docs/{{version}}/configuration#maintenance-mode), since we don't want your tasks to interfere with any unfinished maintenance you may be performing on your server. However, if you would like to force a task to run even in maintenance mode, you may use the `evenInMaintenanceMode` method:

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