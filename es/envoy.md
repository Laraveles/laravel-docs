# Envoy Task Runner

- [Introducción](#introduction) 
    - [Instalación](#installation)
- [Definir tareas](#writing-tasks) 
    - [Configuración](#setup)
    - [Variables](#variables)
    - [Stories](#stories)
    - [Varios servidores](#multiple-servers)
- [Ejecutar tareas](#running-tasks) 
    - [Confirmar la ejecución de la tarea](#confirming-task-execution)
- [Notificaciones](#notifications) 
    - [Slack](#slack)

<a name="introduction"></a>

## Introducción

[Laravel Envoy](https://github.com/laravel/envoy) proporciona una sintaxis minimalista para la definición de las tareas más comunes que se ejecutan en sus servidores remotos. Usando el estilo de sintaxis de Blade, puede configurar fácilmente tareas de despliegue (*deployment*), comandos de Artisan y mucho más. Actualmente, Envoy sólo es compatible con los sistemas operativos Mac y Linux.

<a name="installation"></a>

### Instalación

En primer lugar, instalar Envoy usando el comando `global` de Composer:

    composer global require laravel/envoy
    

Puesto que las librerías globales de Composer pueden causar conflictos de versión de paquetes en algunas ocasiones, se puede considerar usar `cgr`, el cual sería un reemplazo de `composer global require`. Las instrucciones de instalación de la librería `cgr` las puede [encontrar en GitHub](https://github.com/consolidation-org/cgr).

> {note} Asegúrese de situar el directorio `~/.composer/vendor/bin` en su PATH, permitiendo que se encuentre el ejecutable de `envoy` al ejecutar el comando `envoy` en su terminal.

#### Actualizar Envoy

También puede usar Composer para mantener su instalación de Envoy actualizada. Al ejecutar el comando `composer global update` se actualizarán todos sus paquetes Composer instalados globalmente:

    composer global update
    

<a name="writing-tasks"></a>

## Definir tareas

Todas las tareas de Envoy deben definirse en un archivo llamado `Envoy.blade.php` ubicado en la raíz del proyecto. He aquí un ejemplo para empezar:

    @servers(['web' => ['user@192.168.1.1']])
    
    @task('foo', ['on' => 'web'])
        ls -la
    @endtask
    

Como se puede observar, se define un *array* `@servers` al inicio del archivo, lo que le permite hacer referencia a estos servidores en la opción `on` de las declaraciones de tareas. Dentro de las declaraciones `@task`, se debe colocar el código Bash que se deberá ejecutar en el servidor cuando se ejecute la tarea.

Puede forzar un *script* para ejecutarlo localmente especificando la dirección IP del servidor como `127.0.0.1`:

    @servers(['localhost' => '127.0.0.1'])
    

<a name="setup"></a>

### Configuración

A veces, puede que se tenga que ejecutar código PHP antes de ejecutar las tareas en Envoy. Se puede utilizar la directiva ```@setup``` para declarar variables y ejecutar PHP general antes de que se ejecuten cualquiera de sus otras tareas:

    @setup
        $now = new DateTime();
    
        $environment = isset($env) ? $env : "testing";
    @endsetup
    

If you need to require other PHP files before your task is executed, you may use the `@include` directive at the top of your `Envoy.blade.php` file:

    @include('vendor/autoload.php')
    
    @task('foo')
        # ...
    @endtask
    

<a name="variables"></a>

### Variables

If needed, you may pass option values into Envoy tasks using the command line:

    envoy run deploy --branch=master
    

You may access the options in your tasks via Blade's "echo" syntax. Of course, you may also use `if` statements and loops within your tasks. For example, let's verify the presence of the `$branch` variable before executing the `git pull` command:

    @servers(['web' => '192.168.1.1'])
    
    @task('deploy', ['on' => 'web'])
        cd site
    
        @if ($branch)
            git pull origin {{ $branch }}
        @endif
    
        php artisan migrate
    @endtask
    

<a name="stories"></a>

### Stories

Stories group a set of tasks under a single, convenient name, allowing you to group small, focused tasks into large tasks. For instance, a `deploy` story may run the `git` and `composer` tasks by listing the task names within its definition:

    @servers(['web' => '192.168.1.1'])
    
    @story('deploy')
        git
        composer
    @endstory
    
    @task('git')
        git pull origin master
    @endtask
    
    @task('composer')
        composer install
    @endtask
    

Once the story has been written, you may run it just like a typical task:

    envoy run deploy
    

<a name="multiple-servers"></a>

### Multiple Servers

Envoy allows you to easily run a task across multiple servers. First, add additional servers to your `@servers` declaration. Each server should be assigned a unique name. Once you have defined your additional servers, list each of the servers in the task's `on` array:

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])
    
    @task('deploy', ['on' => ['web-1', 'web-2']])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask
    

#### Parallel Execution

By default, tasks will be executed on each server serially. In other words, a task will finish running on the first server before proceeding to execute on the second server. If you would like to run a task across multiple servers in parallel, add the `parallel` option to your task declaration:

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])
    
    @task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask
    

<a name="running-tasks"></a>

## Ejecutar tareas

To run a task or story that is defined in your `Envoy.blade.php` file, execute Envoy's `run` command, passing the name of the task or story you would like to execute. Envoy will run the task and display the output from the servers as the task is running:

    envoy run task
    

<a name="confirming-task-execution"></a>

### Confirming Task Execution

If you would like to be prompted for confirmation before running a given task on your servers, you should add the `confirm` directive to your task declaration. This option is particularly useful for destructive operations:

    @task('deploy', ['on' => 'web', 'confirm' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask
    

<a name="notifications"></a>
<a name="hipchat-notifications"></a>

## Notificaciones

<a name="slack"></a>

### Slack

Envoy también admite el envío de notificaciones a [Slack](https://slack.com) después de ejecutar cada tarea. La directiva `@slack` acepta un URL del *hook* de Slack y un nombre de canal. Puede obtener la URL del *webhook* mediante la creación de una integración "Incoming WebHooks" en su panel de control de Slack. Debe pasar la URL del *webhook* completo a la directiva `@slack`:

    @finished
        @slack('webhook-url', '#bots')
    @endfinished
    

Se puede proporcionar uno de los siguientes como el argumento de canal:

<div class="content-list">
  <ul>
    <li>
      Para enviar una notificación a un canal: <code>#canal</code>
    </li>
    <li>
      Para enviar una notificación a un usuario: <code>@usuario</code>
    </li>
  </ul>
</div>