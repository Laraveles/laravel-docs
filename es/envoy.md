# Envoy Task Runner

- [Introducción](#introduction) 
    - [Instalación](#installation)
- [Definir tareas](#writing-tasks) 
    - [Configuración](#setup)
    - [Variables](#variables)
    - [Historias – *Stories*](#stories)
    - [Varios servidores](#multiple-servers)
- [Ejecutar tareas](#running-tasks) 
    - [Confirmar la ejecución de tareas](#confirming-task-execution)
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
    

Se se precisa de otros ficheros PHP antes de ejecutar la tarea, podrá utilizar la directiva `@include` al inicio de su fichero `Envoy.blade.php`:

    @include('vendor/autoload.php')
    
    @task('foo')
        # ...
    @endtask
    

<a name="variables"></a>

### Variables

Si es necesario, puede pasar valores de opciones a las tareas de Envoy usando la línea de comandos:

    envoy run deploy --branch=master
    

Podrá acceder a las opciones en sus tareas a través de la sintaxis "echo" de Blade. Desde luego, también podrá usar las declaraciones `if` y bucles de sus tareas. Por ejemplo, para verificar la presencia de una variable `$branch` antes de ejecutar el comando `git pull`:

    @servers(['web' => '192.168.1.1'])
    
    @task('deploy', ['on' => 'web'])
        cd site
    
        @if ($branch)
            git pull origin {{ $branch }}
        @endif
    
        php artisan migrate
    @endtask
    

<a name="stories"></a>

### Historias – *Stories*

Las historias agrupan un conjunto de tareas con un nombre único y conveniente, lo que le permite agrupar tareas pequeñas enfocadas en grandes tareas. Por ejemplo, una *story* `deploy` puede ejecutar las tareas `git` y `composer` listando los nombres de las mismas en su definición:

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
    

Una vez que la historia ha sido escrita, puede ejecutarse como una tarea normal:

    envoy run deploy
    

<a name="multiple-servers"></a>

### Varios servidores

Envoy permite ejecutar tareas en varios servidores fácilmente. Para empezar, añada los servidores adicionales a la declaración `@servers`. Cada servidor debe tener un nombre único. Una vez que haya definido sus servidores adicionales, liste cada uno de los servidores en el *array* de tareas `on`:

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])
    
    @task('deploy', ['on' => ['web-1', 'web-2']])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask
    

#### Ejecución paralela

Por defecto, las tareas se ejecutarán sobre cada servidor en serie. En otras palabras, un tarea terminará de correr en el primer servidor antes de proceder a ejecutarla en el segundo. Si prefiere ejecutar la tarea en varios servidores en paralelo, agregue `parallel` a la declaración:

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])
    
    @task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask
    

<a name="running-tasks"></a>

## Ejecutar tareas

Para ejecutar una tarea o historia definida en el fichero `Envoy.blade.php`, ejecute el comando de Envoy `run`, pasando el nombre de la tarea o historia a ejecutar. Envoy ejecutará la tarea y mostrará la salida de los servidores donde se está corriendo:

    envoy run task
    

<a name="confirming-task-execution"></a>

### Confirmar la ejecución de tareas

Si desea que se le solicite confirmación antes de correr una tarea determinada en sus servidores, debe agregar la directiva `confirm` a su declaración de tarea. Esta opción es particularmente útil para operaciones destructivas:

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