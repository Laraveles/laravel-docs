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
- [Programando la ejecución de comandos](#programmatically-executing-commands) 
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

### Comandos definidos como *funciones anónimas* (Closure)

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
    

#### Descripciones de comandos definidos como funciones anónimas

Cuando se define un comando a través de una función anónima, usted puede usar el método `describe` para añadir la descripción al comando. Esta descripción será mostrada cuando ejecuten los comandos `php artisan list` o `php artisan help`:

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    })->describe('Build the project');
    

<a name="defining-input-expectations"></a>

## Definición de las entradas esperadas

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

#### Atajos para las opciones

Para asignar un atajo cuando se define una opción, usted puede especificarla antes del nombre de la opción y utilizando el delimitador | para separar el atajo del nombre completo de la opción:

    email:send {user} {--Q|queue}
    

<a name="input-arrays"></a>

### Matrices de entrada

Si quisiera definir argumentos u opciones que esperan una entrada en forma de matriz, puedes usar el carácter `*`. Primero, veamos un ejemplo que especifica una matriz como argumento:

    email:send {user*}
    

Cuando se llame este método, los argumentos de `user` pueden ser pasados, en orden, a la línea de comandos. Por ejemplo, el siguiente comando asignará `['foo', 'bar']` como valor de `user`:

    php artisan email:send foo bar
    

Cuando definimos una opción que espera una matriz como entrada, cada valor de opción pasado al comando debería ser precedido por el nombre de opción:

    email:send {user} {--id=*}
    
    php artisan email:send --id=1 --id=2
    

<a name="input-descriptions"></a>

### Descripciones de la entrada

Usted puede asignar descripciones a argumentos y opciones de entrada separando el parámetro de la descripción usando los dos puntos (:). Si necesita un poco más de espacio para definir su comando, siéntase libre de separar la definición en varias líneas:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send
                            {user : The ID of the user}
                            {--queue= : Whether the job should be queued}';
    

<a name="command-io"></a>

## Comandos de I/O (E/S)

<a name="retrieving-input"></a>

### Obtener la entrada

Mientras que se está ejecutándo el comando, usted puede, obviamente, acceder a los valores de los argumentos y opciones aceptados por su comando. Para ello, usted puede utilizar los métodos `argument` y `option`:

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
    

Las opciones pueden ser obtenidas tan fácilmente como los argumentos usando el método `option`. Para obtener las opciones como una matriz, llama al método `options`:

    // Retrieve a specific option...
    $queueName = $this->option('queue');
    
    // Retrieve all options...
    $options = $this->options();
    

Si el argumento u opción no existe, se retornará `null`.

<a name="prompting-for-input"></a>

### Solicitud de los parámetros de entrada

Además de mostrar la salida, usted puede, además, solicitar información al usuario durante la ejecución de su comando. El método `ask` preguntará al usuario por información, aceptará sus entradas y entregará esta información al comando:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('What is your name?');
    }
    

El método `secret` es parecido a `ask`, pero la entrada del usuario no será visible mientras se escribe en la consola. Este método resulta útil cuando se solicita información sensible como contraseñas:

    $password = $this->secret('What is the password?');
    

#### Solicitar confirmación

Si se necesita pedirle al usuario una simple confirmación, puede utilizar el método `confirm`. Por defecto, este método retornará `false`. Si embargo, si el usuario introduce `y` o `yes` como respuesta a la solicitud desde la consola de comandos, el método retornará `true`.

    if ($this->confirm('Do you wish to continue?')) {
        //
    }
    

#### Autocompletado

El método `anticipate` puede ser usado para proveer autocompletado para posibles elecciones. El usuario podrá elegir cualquier respuesta, independientemente de las opciones que se le ofrecen como autocompletado:

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);
    

#### Preguntas de elección múltiple

Si necesita dar al usuario un conjunto predeterminado de elecciones, puede usar el método `choice`. Puede especificar el valor por defecto para ser retornado si ninguna opción es seleccionada:

    $name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $default);
    

<a name="writing-output"></a>

### Generar salida

Para enviar contenido de la salida a la consola, utilice los métodos `line`, `info`, `comment`, `question` y `error`. Cada uno de estos métodos usará colores ANSI apropiados para su cometido. Por ejemplo, permite mostrar información general al usuario. Normalmente, el método `info` mostrará texto verde en la consola:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('Display this on the screen');
    }
    

Para mostrar un mensaje de error, utilice el método `error`. El mensaje de error, normalmente, será mostrado en rojo:

    $this->error('Something went wrong!');
    

Si desearía mostrar texto plano, sin colorear en la salida de la consola, utilice el método `line`:

    $this->line('Display this on the screen');
    

#### Diseño de tablas

El método `table` facilita el correcto formato de datoe en múltiples filas / columnas. Simplemente pasa los encabezados y filas al método. El ancho y alto será calculado de forma dinámica en función de los datos:

    $headers = ['Name', 'Email'];
    
    $users = App\User::all(['name', 'email'])->toArray();
    
    $this->table($headers, $users);
    

#### Barras de progreso

Para tareas largas, podría ser útil mostrar indicadores de progreso. Utilizando el objeto de salida, podemos comenzar, avanzar y detener la barra de progreso. Primero, define el número total de pasos que recorrer el proceso. Entonces, avance la barra de progreso tras procesar cada paso:

    $users = App\User::all();
    
    $bar = $this->output->createProgressBar(count($users));
    
    foreach ($users as $user) {
        $this->performTask($user);
    
        $bar->advance();
    }
    
    $bar->finish();
    

Para opciones más avanzadas, eche un vistazo a la [documentación del componente de la Barra de Progreso de Symfony](https://symfony.com/doc/2.7/components/console/helpers/progressbar.html).

<a name="registering-commands"></a>

## Registro de comandos

Debido a que el método `load` es llamado en el método `commands` del *kernel* de la consola, todos los comandos en el directorio `app/Console/Commands` serán registrados automáticamente en Artisan. De hecho, usted es libre de hacer llamadas adicionales al método `load` para escanear otros directorios donde tenga comandos para añadir a Artisan:

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
    

Puede también registrar comandos manualmente añadiendo el nombre de la clase a la propiedad `$commands` del archivo `app/Console/Kernel.php`. Cuando Artisan arranca, todos los comandos enumerados en esta propiedad serán resueltos por el [service container](/docs/{{version}}/container) y registrados con Artisan:

    protected $commands = [
        Commands\SendEmails::class
    ];
    

<a name="programmatically-executing-commands"></a>

## Programando la ejecución de comandos

A veces puede desear que se ejecute un comando Artisan desde fuera de la consola CLI. Por ejemplo, puede disparar un comando Artisan desde una ruta o controlador. Se puede hacer utilizando el método `call` de la facade `Artisan`. El método `call` acepta el nombre del comando como primer argumento y una matriz de parámetros como segundo argumento. El código de salida será retornado:

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);
    
        //
    });
    

Usando el método `queue` de la *facade* `Artisan`, puede incluso poner en cola comandos de Artisan que serán procesados en segundo plano por sus [procesadores de colas](/docs/{{version}}/queues). Antes de usar este método, asegúrese de tener configurado su *cola de trabajos* y de que esté corriendo un procesador de colas:

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);
    
        //
    });
    

Puede también especificar la conexión o cola a la que el comando de Artisan debe ser lanzado:

    Artisan::queue('email:send', [
        'user' => 1, '--queue' => 'default'
    ])->onConnection('redis')->onQueue('commands');
    

#### Pasar matrices como valor

Si su comando define una opción que acepta una matriz, debe simplemente pasar una matriz de valores a esa opción:

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--id' => [5, 13]
        ]);
    });
    

#### Pasar valores *booleanos*

Si necesita especificar el valor de una opción que no acepta cadenas de texto como valor, tal como el marcador `--force` en el comando `migrate:refresh` usted debería pasar `true` o `false`:

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);
    

<a name="calling-commands-from-other-commands"></a>

### Ejecutar Comandos desde otros comandos

En ocasiones puede requerir llamar otros comandos desde un comando Artisan existente. Usted puede hacerlo utilizando el método `call`. Este método `call` acepta el nombre del comando y una matriz de parámetros del comando:

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
    

Si quisiera llamar a otro comando y eliminar cualquier salida que pueda generar, puede utilizar el método `callSilent`. El método `callSilent` tiene la misma firma que el método `call`:

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);