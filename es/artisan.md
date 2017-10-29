# Consola Artisan

- [Introducción](#introduction)
- [Crear comandos](#writing-commands) 
    - [Generar comandos](#generating-commands)
    - [Estructura de un comando](#command-structure)
    - [Comandos en funciones anónimas (closure)](#closure-commands)
- [Definición de expectativas de entrada](#defining-input-expectations) 
    - [Argumentos](#arguments)
    - [Opciones](#options)
    - [Matrices de entradas](#input-arrays)
    - [Descripciones de entrada](#input-descriptions)
- [E/S del comando](#command-io) 
    - [Obtener la entrada](#retrieving-input)
    - [Solicitud de parámetros de entrada](#prompting-for-input)
    - [Generar la salida](#writing-output)
- [Registro de los comandos](#registering-commands)
- [Programmatically Executing Commands](#programmatically-executing-commands) 
    - [Ejecutar Comandos desde otros comandos](#calling-commands-from-other-commands)

<a name="introduction"></a>

## Introducción

Artisan es el nombre de la interfaz de línea de comandos incluída en Laravel. Ésta proporciona una serie de comandos útiles que podrán ayudar mientras desarrolla su aplicación. Para conocer la lista de todos los comandos Artisan disponibles, usted puede utilizar el comando `list`:

    php artisan list
    

Cada comando incluye además una ayuda en pantalla que muestra y describe los parámetros y argumentos disponibles para el comando. Para visualizar la ayuda en pantalla, simplemente precede el nombre del comando con `help`:

    php artisan help migrate
    

#### Laravel REPL

Todas las aplicaciones Laravel incluyen *Tinker*, una consola (REPL) utilizando el motor del paquete [PsySH](https://github.com/bobthecow/psysh). Tinker le permite interactuar con toda la aplicación Laravel a través de la linea de comandos, incluyendo *Eloquent ORM*, trabajos, eventos, y más. Para entrar en el entorno Tinker ejecute el comando Artisan `tinker`:

    php artisan tinker
    

<a name="writing-commands"></a>

## Escribir comandos

Además de los comandos que provee Artisan, usted puede crear también sus propios comandos personalizados. Los comandos se almacenan normalmente en el directorio `app/Console/Commands`; sin embargo, usted es libre de elegir su propia ubicación de almacenamiento siempre y cuando sus comandos puedan ser cargados por *Composer*.

<a name="generating-commands"></a>

### Generar comandos

Para crear un nuevo comando utilice el comando de Artisan `make:command`. Este comando creará una nueva clase *command* en el directorio `app/Console/Commands`. No se preocupe si este directorio no existe en la aplicación, ya que éste se creará la primera vez que ejecute el comando Artisan `make:command`. El comando generado incluirá el conjunto predeterminado de propiedades y métodos que están presentes en todos los comandos:

    php artisan make:command SendEmails
    

<a name="command-structure"></a>

### Estructura de un comando

Tras generar el comando usted debería rellenar las propiedades `signature` y `description` de la clase, las cuales serán usadas cuando se muestre el comando en la lista que obtenemos al ejecutar el comando Artisan `list`. El método `handle` será llamado cuando el comando es ejecutado. Puede colocar la lógica del comando dentro de este método.

> {tip} Para facilitar la reutilización de código, es una buena práctica mantener los comandos de consola ligeros y permitir delegar la lógica para cumplir sus tareas a los servicios de la aplicación. En el ejemplo que mostramos más abajo se inyecta una clase servicio para hacer la "parte pesada" del envío de los correos electrónicos.

Veamos un comando de ejemplo. Tener en cuenta que somos capaces de inyectar cualquier dependencia que necesitemos dentro del constructor del comando. El [*service container*](/docs/{{version}}/container) de Laravel inyectará automáticamente, en en el constructor, todas las dependencias que estén cualificadas por su tipo:

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

Los comandos basadoe en una función anónima (*Closure*) son una alternativa a definir comandos de consola como clases. Del mismo modo que las rutas definida como funciones anónimas son la alternativa al uso de controladores, piense que los comandos definidos así son una alternativa a los comandos definidos en una clase. En el interior del método `commands` del fichero `app/Console/Kernel.php`, Laravel carga el fichero `routes/console.php`:

    /**
     * Register the Closure based commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        require base_path('routes/console.php');
    }
    

Aunque este fichero no define rutas HTTP, éstas definen puntos de entrada a la consola (rutas) dentro de la aplicación. En el interior de este fichero se pueden definir todos los comandos definidos como funciones anónimas usando el método `Artisan::command`. El método `command` acepta dos argumentos: La [firma del comando](#defining-input-expectations) y una función anónima que recibe los argumentos y opciones del comando:

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    });
    

La función anónima está enlazada con la instancia de un comando, por tanto, tiene acceso total a todos los métodos de ayuda que tendría normalmente accesibles en una clase de comando.

#### Sugerencias de tipo para las dependencias

Además de recibir los argumentos y opciones del comando, las funciones anónimas pueden también establecer los tipos para las dependencias adicionales que quisieran ser resueltas a través del [service container](/docs/{{version}}/container):

    use App\User;
    use App\DripEmailer;
    
    Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
        $drip->send(User::find($user));
    });
    

#### Closure Command Descriptions

Cuando se define un comando a través de una función anónima, usted puede usar el método `describe` para añadir la descripción al comando. Esta descripción será mostrada cuando ejecuten los comandos `php artisan list` o `php artisan help`:

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    })->describe('Build the project');
    

<a name="defining-input-expectations"></a>

## Defining Input Expectations

Al escribir comandos de consola, es bastante común obtener datos de entrada desde el usuario a través de argumentos u opciones. Laravel simplifica la definición de las entradas requeridas por el usuario utilizando la propiedad `signature` del comando. La propiedad `signature` le permite definir el nombre, argumentos y opciones del comando con una sintaxis simple y expresiva, similar a la de la rutas.

<a name="arguments"></a>

### Argumentos

Todos los argumentos y opciones introducidos por el usuario irán entre llaves. En el siguiente ejemplo, el comando define un argumento obligatorio **required**: `user`:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user}';
    

Puede crear argumentos opcionales y definir valores por defecto para los argumentos:

    // Optional argument...
    email:send {user?}
    
    // Optional argument with default value...
    email:send {user=foo}
    

<a name="options"></a>

### Opciones

Las opciones, al igual que los argumentos, son otra forma de recoger valores de entrada aportados por el usuario. Las opciones son precedidas por dos guiones (`--`) cuando se especifican en la línea de comandos. Hay dos tipos de opciones: aquellas que reciben un valor y las que no. Las opciones que no reciben un valor funcionan como un "interuptor" lógico. Mire el siguiente ejemplo de este tipo de opción:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';
    

En este ejemplo, la opción `--queue` se puede especificar directamente cuando se ejecuta el comando Artisan. Si se pasa el opción `--queue`, el valor de la opción será `true`. De lo contrario, el valor será `false`:

    php artisan email:send 1 --queue
    

<a name="options-with-values"></a>

#### Opciones con Valores

Ahora, veamos una opción que espera un valor. Si el usuario debe especificar un valor para una opción, ponga `=` como sufijo de la opción:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';
    

En este ejemplo, el usuario puede pasar un valor para la opción:

    php artisan email:send 1 --queue=default
    

Puede asignar valores por defecto a las opciones especificándolo después del nombre de la opción. Si el usuario no se pasa ningún valor se usará el el valor por defecto:

    email:send {user} {--queue=default}
    

<a name="option-shortcuts"></a>

#### Option Shortcuts

Para asignar un atajo cuando se define una opción, usted puede especificarla antes del nombre de la opción y utilizando el delimitador | para separar el atajo del nombre completo de la opción:

    email:send {user} {--Q|queue}
    

<a name="input-arrays"></a>

### Input Arrays

Si quisiera definir argumentos u opciones que esperan una entrada en forma de matriz, puedes usar el carácter `*`. Primero, veamos un ejemplo que especifica una matriz como argumento:

    email:send {user*}
    

Cuando se llame este método, los argumentos de `user` pueden ser pasados, en orden, a la línea de comandos. Por ejemplo, el siguiente comando asignará `['foo', 'bar']` como valor de `user`:

    php artisan email:send foo bar
    

Cuando definimos una opción que espera una matriz como entrada, cada valor de opción pasado al comando debería ser precedido por el nombre de opción:

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
    

Si necesita obtener todos los argumentos como una `matriz`, use el método `arguments`:

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