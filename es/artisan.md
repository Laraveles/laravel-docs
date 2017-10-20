# Consola Artisan

- [Introducción](#introduction)
- [Crear Comandos](#writing-commands) 
    - [Generar Comandos](#generating-commands)
    - [Estructura del Comando](#command-structure)
    - [Comandos Closure](#closure-commands)
- [Definición de Expectativas de Entrada](#defining-input-expectations) 
    - [Argumentos](#arguments)
    - [Opciones](#options)
    - [Arrays de Entrada](#input-arrays)
    - [Descripciones de Entrada](#input-descriptions)
- [E/S del Comando](#command-io) 
    - [Obtener Input](#retrieving-input)
    - [Solicitud de Parámetros de Entrada](#prompting-for-input)
    - [Generar Salida](#writing-output)
- [Registro de Comandos](#registering-commands)
- [Programmatically Executing Commands](#programmatically-executing-commands) 
    - [Ejecutar Comandos Desde Otros Comandos](#calling-commands-from-other-commands)

<a name="introduction"></a>

## Introducción

Artisan es el nombre de la interfaz de línea de comandos incluida en Laravel. Proporciona una serie de comandos útiles que podrán ayudar mientras se desarrolla una aplicación. Para conocer todos los comandos Artisan disponibles, utilizar el comando `list`:

    php artisan list
    

Cada comando incluye además una pantalla de "help" que muestra y describe los parámetros y opciones disponibles en el comando. Para visualizar la pantalla de ayuda, simplemente preceder el nombre del comando con `help`:

    php artisan help migrate
    

#### Laravel REPL

Todas las aplicaciones Laravel incluyen Tinker, un REPL impulsado por el páquete [PsySH](https://github.com/bobthecow/psysh). Tinker permite interactuar con toda la aplicación a través de la linea de comandos, incluyendo Eloquent ORM, jobs, eventos, y más. Para entrar en el entorno de `tinker` ejecutar el comando Artisan:

    php artisan tinker
    

<a name="writing-commands"></a>

## Crear Comandos

Además de los comandos que provee Artisan, se pueden crear comandos propios personalizados. Estos comandos se almacenan normalmente en el directorio `app/Console/Commands`; sin embargo, se es libre de elegir su propia ubicación de almacenamiento siempre y cuando sus comandos puedan ser cargados por Composer.

<a name="generating-commands"></a>

### Generar Comandos

Para crear un nuevo comando se utiliza el comando de Artisan `make:command`. Este comando creará una nueva clase *command* en el directorio `app/Console/Commands`. No se preocupe si este directorio no existe en la aplicación, ya que se creará la primera vez que se ejecute el comando Artisan `make:command`. El comando generado incluirá el conjunto predeterminado de propiedades y métodos que están presentes en todos los comandos:

    php artisan make:command SendEmails
    

<a name="command-structure"></a>

### Estructura de un Comando

Tras generar el comando debería rellenar las propiedades `signature` y `description` de la clase que serán usadas al mostrar el comando en la lista que obtenemos al ejecutar el comando Artisan `list`. El método `handle` será llamado cuando el comando sea ejecutado. Puede colocar la lógica del comando dentro de este método.

> {tip} Para facilitar la reutilización de código, es buena práctica mantener los comandos de consola ligeros y delegar la lógica a servicios de aplicación. En el ejemplo que mostramos más abajo se inyecta un servicio para hacer la "parte pesada" de enviar correos electrónicos.

Veamos un comando de ejemplo. Tener en cuenta que es posible inyectar cualquier dependencia necesaria en el constructor del comando. El [service container](/docs/{{version}}/container) de Laravel inyectará automáticamente todas las dependencias que se le indiquen como sugerencia de tipo en en el constructor:

    <?php
    
    namespace App\Console\Commands;
    
    use App\User;
    use App\DripEmailer;
    use Illuminate\Console\Command;
    
    class SendEmails extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'email:send {user}';
    
        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Send drip e-mails to a user';
    
        /**
         * The drip e-mail service.
         *
         * @var DripEmailer
         */
        protected $drip;
    
        /**
         * Create a new command instance.
         *
         * @param  DripEmailer  $drip
         * @return void
         */
        public function __construct(DripEmailer $drip)
        {
            parent::__construct();
    
            $this->drip = $drip;
        }
    
        /**
         * Execute the console command.
         *
         * @return mixed
         */
        public function handle()
        {
            $this->drip->send(User::find($this->argument('user')));
        }
    }
    

<a name="closure-commands"></a>

### Closure Commands

Los comandos definidos como una *Closure* (función anónima) presentan una alternativa a definir comandos de consola como clases. Del mismo modo que las rutas definida como *Closures* son una alternativa al uso de controladores, piense que los comandos definidos como *Closures* son una alternativa a los comandos definidos en una clase. En el interior del método `commands` del fichero `app/Console/Kernel.php`, Laravel carga el fichero `routes/console.php`:

    /**
     * Register the Closure based commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        require base_path('routes/console.php');
    }
    

Aunque este fichero no define rutas HTTP aquí se definen puntos de entrada a la consola (rutas) dentro de la aplicación. En el interior de este fichero se pueden definir todos los comandos definidos como *Closure* usando el método `Artisan::command`. El método `command` acepta dos parámetros: La [firma del comando](#defining-input-expectations) y una *Closure* que recibirá los parámetros y opciones del comando:

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    });
    

La *Closure* está enlazada con la instancia de un comando, por tanto, tiene acceso a todos los métodos *helper* que tendría accesibles en una clase de comando.

#### Type-Hinting Dependencies

Además de recibir los parámetros y opciones de su comando, en los comandos definidos como *Closure* puede también usar las sugerencias de tipo para cargar dependencias a través del [service container](/docs/{{version}}/container):

    use App\User;
    use App\DripEmailer;
    
    Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
        $drip->send(User::find($user));
    });
    

#### Closure Command Descriptions

When defining a Closure based command, you may use the `describe` method to add a description to the command. This description will be displayed when you run the `php artisan list` or `php artisan help` commands:

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    })->describe('Build the project');
    

<a name="defining-input-expectations"></a>

## Defining Input Expectations

When writing console commands, it is common to gather input from the user through arguments or options. Laravel makes it very convenient to define the input you expect from the user using the `signature` property on your commands. The `signature` property allows you to define the name, arguments, and options for the command in a single, expressive, route-like syntax.

<a name="arguments"></a>

### Arguments

All user supplied arguments and options are wrapped in curly braces. In the following example, the command defines one **required** argument: `user`:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user}';
    

You may also make arguments optional and define default values for arguments:

    // Optional argument...
    email:send {user?}
    
    // Optional argument with default value...
    email:send {user=foo}
    

<a name="options"></a>

### Options

Options, like arguments, are another form of user input. Options are prefixed by two hyphens (`--`) when they are specified on the command line. There are two types of options: those that receive a value and those that don't. Options that don't receive a value serve as a boolean "switch". Let's take a look at an example of this type of option:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';
    

In this example, the `--queue` switch may be specified when calling the Artisan command. If the `--queue` switch is passed, the value of the option will be `true`. Otherwise, the value will be `false`:

    php artisan email:send 1 --queue
    

<a name="options-with-values"></a>

#### Options With Values

Next, let's take a look at an option that expects a value. If the user must specify a value for an option, suffix the option name with a `=` sign:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';
    

In this example, the user may pass a value for the option like so:

    php artisan email:send 1 --queue=default
    

You may assign default values to options by specifying the default value after the option name. If no option value is passed by the user, the default value will be used:

    email:send {user} {--queue=default}
    

<a name="option-shortcuts"></a>

#### Option Shortcuts

To assign a shortcut when defining an option, you may specify it before the option name and use a | delimiter to separate the shortcut from the full option name:

    email:send {user} {--Q|queue}
    

<a name="input-arrays"></a>

### Input Arrays

If you would like to define arguments or options to expect array inputs, you may use the `*` character. First, let's take a look at an example that specifies an array argument:

    email:send {user*}
    

When calling this method, the `user` arguments may be passed in order to the command line. For example, the following command will set the value of `user` to `['foo', 'bar']`:

    php artisan email:send foo bar
    

When defining an option that expects an array input, each option value passed to the command should be prefixed with the option name:

    email:send {user} {--id=*}
    
    php artisan email:send --id=1 --id=2
    

<a name="input-descriptions"></a>

### Input Descriptions

You may assign descriptions to input arguments and options by separating the parameter from the description using a colon. If you need a little extra room to define your command, feel free to spread the definition across multiple lines:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send
                            {user : The ID of the user}
                            {--queue= : Whether the job should be queued}';
    

<a name="command-io"></a>

## Command I/O

<a name="retrieving-input"></a>

### Retrieving Input

While your command is executing, you will obviously need to access the values for the arguments and options accepted by your command. To do so, you may use the `argument` and `option` methods:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $userId = $this->argument('user');
    
        //
    }
    

If you need to retrieve all of the arguments as an `array`, call the `arguments` method:

    $arguments = $this->arguments();
    

Options may be retrieved just as easily as arguments using the `option` method. To retrieve all of the options as an array, call the `options` method:

    // Retrieve a specific option...
    $queueName = $this->option('queue');
    
    // Retrieve all options...
    $options = $this->options();
    

If the argument or option does not exist, `null` will be returned.

<a name="prompting-for-input"></a>

### Prompting For Input

In addition to displaying output, you may also ask the user to provide input during the execution of your command. The `ask` method will prompt the user with the given question, accept their input, and then return the user's input back to your command:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('What is your name?');
    }
    

The `secret` method is similar to `ask`, but the user's input will not be visible to them as they type in the console. This method is useful when asking for sensitive information such as a password:

    $password = $this->secret('What is the password?');
    

#### Asking For Confirmation

If you need to ask the user for a simple confirmation, you may use the `confirm` method. By default, this method will return `false`. However, if the user enters `y` or `yes` in response to the prompt, the method will return `true`.

    if ($this->confirm('Do you wish to continue?')) {
        //
    }
    

#### Auto-Completion

The `anticipate` method can be used to provide auto-completion for possible choices. The user can still choose any answer, regardless of the auto-completion hints:

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);
    

#### Multiple Choice Questions

If you need to give the user a predefined set of choices, you may use the `choice` method. You may set the default value to be returned if no option is chosen:

    $name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $default);
    

<a name="writing-output"></a>

### Writing Output

To send output to the console, use the `line`, `info`, `comment`, `question` and `error` methods. Each of these methods will use appropriate ANSI colors for their purpose. For example, let's display some general information to the user. Typically, the `info` method will display in the console as green text:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('Display this on the screen');
    }
    

To display an error message, use the `error` method. Error message text is typically displayed in red:

    $this->error('Something went wrong!');
    

If you would like to display plain, uncolored console output, use the `line` method:

    $this->line('Display this on the screen');
    

#### Table Layouts

The `table` method makes it easy to correctly format multiple rows / columns of data. Just pass in the headers and rows to the method. The width and height will be dynamically calculated based on the given data:

    $headers = ['Name', 'Email'];
    
    $users = App\User::all(['name', 'email'])->toArray();
    
    $this->table($headers, $users);
    

#### Progress Bars

For long running tasks, it could be helpful to show a progress indicator. Using the output object, we can start, advance and stop the Progress Bar. First, define the total number of steps the process will iterate through. Then, advance the Progress Bar after processing each item:

    $users = App\User::all();
    
    $bar = $this->output->createProgressBar(count($users));
    
    foreach ($users as $user) {
        $this->performTask($user);
    
        $bar->advance();
    }
    
    $bar->finish();
    

For more advanced options, check out the [Symfony Progress Bar component documentation](https://symfony.com/doc/2.7/components/console/helpers/progressbar.html).

<a name="registering-commands"></a>

## Registering Commands

Because of the `load` method call in your console kernel's `commands` method, all commands within the `app/Console/Commands` directory will automatically be registered with Artisan. In fact, you are free to make additional calls to the `load` method to scan other directories for Artisan commands:

    /**
     * Register the commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');
        $this->load(__DIR__.'/MoreCommands');
    
        // ...
    }
    

You may also manually register commands by adding its class name to the `$command` property of your `app/Console/Kernel.php` file. When Artisan boots, all the commands listed in this property will be resolved by the [service container](/docs/{{version}}/container) and registered with Artisan:

    protected $commands = [
        Commands\SendEmails::class
    ];
    

<a name="programmatically-executing-commands"></a>

## Programmatically Executing Commands

Sometimes you may wish to execute an Artisan command outside of the CLI. For example, you may wish to fire an Artisan command from a route or controller. You may use the `call` method on the `Artisan` facade to accomplish this. The `call` method accepts the name of the command as the first argument, and an array of command parameters as the second argument. The exit code will be returned:

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);
    
        //
    });
    

Using the `queue` method on the `Artisan` facade, you may even queue Artisan commands so they are processed in the background by your [queue workers](/docs/{{version}}/queues). Before using this method, make sure you have configured your queue and are running a queue listener:

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);
    
        //
    });
    

You may also specify the connection or queue the Artisan command should be dispatched to:

    Artisan::queue('email:send', [
        'user' => 1, '--queue' => 'default'
    ])->onConnection('redis')->onQueue('commands');
    

#### Passing Array Values

If your command defines an option that accepts an array, you may simply pass an array of values to that option:

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--id' => [5, 13]
        ]);
    });
    

#### Passing Boolean Values

If you need to specify the value of an option that does not accept string values, such as the `--force` flag on the `migrate:refresh` command, you should pass `true` or `false`:

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);
    

<a name="calling-commands-from-other-commands"></a>

### Calling Commands From Other Commands

Sometimes you may wish to call other commands from an existing Artisan command. You may do so using the `call` method. This `call` method accepts the command name and an array of command parameters:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);
    
        //
    }
    

If you would like to call another console command and suppress all of its output, you may use the `callSilent` method. The `callSilent` method has the same signature as the `call` method:

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);