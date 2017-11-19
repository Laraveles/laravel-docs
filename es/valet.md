# Laravel Valet

- [Introducción](#introduction) 
    - [Valet o Homestead](#valet-or-homestead)
- [Instalación](#installation) 
    - [Actualizar](#upgrading)
- [Servir sitios](#serving-sites) 
    - [El comando "park"](#the-park-command)
    - [El comando "link"](#the-link-command)
    - [Asegurar sitios con TLS](#securing-sites)
- [Compartir sitios](#sharing-sites)
- [Drivers de Valet personalizados](#custom-valet-drivers) 
    - [Drivers locales](#local-drivers)
- [Otros comandos de Valet](#other-valet-commands)

<a name="introduction"></a>

## Introducción

Valet es un entorno de desarrollo para minimalistas de Mac. No Vagrant, sin archivo `/etc/hosts`. Incluso se pueden compartir los sitios de forma pública utilizando tunes locales. *Sí, también nos gusta.*

Laravel Valet configura su Mac para ejecutar [Nginx](https://www.nginx.com/) en segundo plano cuando la máquina arranca. Entonces, utilizando [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq), Valet crea un *proxy* de todas las peticiones sobre el dominio `*.dev` para apuntar a los sitios instalados en su máquina.

En otras palabras, un entorno de desarrollo tremendamente potente que utiliza únicamente 7 MB de RAM. Valet no es un reemplazo para Vagrant o Homestead, sino una alternativa si se prefieren servicios básicos, alta velocidad o se está trabajando en una máquina con RAM limitada.

Por defecto, Valet soporta, pero no limitado a:

<div class="content-list">
  <ul>
    <li>
      <a href="https://laravel.com">Laravel</a>
    </li>
    <li>
      <a href="https://lumen.laravel.com">Lumen</a>
    </li>
    <li>
      <a href="https://roots.io/bedrock/">Bedrock</a>
    </li>
    <li>
      <a href="https://cakephp.org">CakePHP 3</a>
    </li>
    <li>
      <a href="http://www.concrete5.org/">Concrete5</a>
    </li>
    <li>
      <a href="https://contao.org/en/">Contao</a>
    </li>
    <li>
      <a href="https://craftcms.com">Craft</a>
    </li>
    <li>
      <a href="https://www.drupal.org/">Drupal</a>
    </li>
    <li>
      <a href="http://jigsaw.tighten.co">Jigsaw</a>
    </li>
    <li>
      <a href="https://www.joomla.org/">Joomla</a>
    </li>
    <li>
      <a href="https://github.com/themsaid/katana">Katana</a>
    </li>
    <li>
      <a href="https://getkirby.com/">Kirby</a>
    </li>
    <li>
      <a href="https://magento.com/">Magento</a>
    </li>
    <li>
      <a href="https://octobercms.com/">OctoberCMS</a>
    </li>
    <li>
      <a href="https://sculpin.io/">Sculpin</a>
    </li>
    <li>
      <a href="https://www.slimframework.com">Slim</a>
    </li>
    <li>
      <a href="https://statamic.com">Statamic</a>
    </li>
    <li>
      HTML estático
    </li>
    <li>
      <a href="https://symfony.com">Symfony</a>
    </li>
    <li>
      <a href="https://wordpress.org">WordPress</a>
    </li>
    <li>
      <a href="https://framework.zend.com">Zend</a>
    </li>
  </ul>
</div>

Sin embargo, se puede extender Valet con [drivers propios](#custom-valet-drivers).

<a name="valet-or-homestead"></a>

### Valet o Homestead

Como ya puede que sepa, Laravel ofrece [Homestead](/docs/{{version}}/homestead), otro entorno de desarrollo local. Homestead y Valet difieren en la audiencia y en el enfoque. Homestead incluye una máquina virtual Ubuntu completa con configuración Nginx automática. Homestead es una buena elección si se quiere tener una máquina Linux virtualizada como entorno de desarrollo o se está en Windows o Linux.

Valet únicamente soporta Mac y requiere instalar PHP y un servidor de base de datos directamente en la máquina local. Esto es muy sencillo utilizando [Homebrew](http://brew.sh/) con comandos como `brew install php71` y `bre install mysql`. Valet provee de un rapidísimo entorno de desarrollo con un consumo de recursos mínimo, por lo que es bueno para desarrolladores quienes únicamente necesitan PHP / MySQL y no necesitan un entorno virtualizado completo.

Ambos son buenas opciones para configurar el entorno de desarrollo local. Cual elegir ya es decisión personal de cada individuo o equipo dependiendo de sus necesidades.

<a name="installation"></a>

## Instalación

**Valet require macOS y [Homebrew](http://brew.sh/). Antes de instalar, hay que asegurarse de no tener otros programas como Apache o Nginx enlazados al puerto 80.**

<div class="content-list">
  <ul>
    <li>
      Instalar o actualizar <a href="http://brew.sh/">Homebrew</a> a la última versión con <code>brew update</code>.
    </li>
    <li>
      Instalar PHP 7.1 utilizando Hombrew <code>brew install homebrew/php/php71</code>.
    </li>
    <li>
      Instalar Valet con Composer vía <code>composer global require laravel/valet</code>. Asegurarse que el directorio <code>~/.composer/vendor/bin</code> está en el "PATH" del sistema.
    </li>
    <li>
      Ejecutar el comando <code>valet install</code>. Esto configurará e instalará Valet y DnsMasq y registrará el <em>daemon</em> para que Valet se ejecute cuando el sistema arranca.
    </li>
  </ul>
</div>

Una vez que Valet se ha instalado, pruebe a hacer *ping* a cualquier dominio `*.dev` desde la terminal `ping foobar.dev`. Si Valet está correctamente instalado se debería ver que este dominio responde `127.0.0.1`.

Valet arrancará su *daemon* cada vez que la máquina inicie. No es necesario ejecutar `valet start` o `valet install` de nuevo una vez que la instalación de Valet esté completa.

#### Utilizar Otro Dominio

Por defecto, Valet sirve los proyecto sutilizando el TLD `.dev`. Si se desea utilizar otro dominio, se puede hacer utilizando el comando `valet domain nombre-tld`.

Por ejemplo, para utilizar el dominio `.app` en lugar de `.dev`, ejecutar `valet domain app` y Valet comenzará a servir los proyectos bajo `*.app` de forma automática.

#### Base de datos

Si se requiere de una base de datos, se puede instalar MySQL a través de la linea de comandos `brew install mysql`. Una vez que MySQL esté instalado, se puede iniciar con el comando `brew services start mysql`. Se puede entonces conectar a la base de datos `127.0.0.1` con usuario `root` y sin contraseña.

<a name="upgrading"></a>

### Actualizar

Se puede actualizar la instalación de Valet utilizando el comando `composer global update`. Después de actualizar, es una buena práctica ejecutar el comando `valet install` para que Valet pueda ejecutar actualizaciones adicionales a los archivos de configuración si fuera necesario.

#### Actualizar a Valet 2.0

Valet 2.0 reemplaza el servidor web subyacente de Caddy a Nginx. Antes de actualizar a esta versión se deben ejecutar los siguientes comandos para detener y desinstalar el <0>daemon</0> Caddy existente:

    valet stop
    valet uninstall
    

A continuación, se debe actualizar a la última versión de Valet. Dependiendo de como se haya instalado Valet, se realizará esta acción a través de Git o Composer. Si se instaló a través de Composer, se debe utilizar el siguiente comando para actualizar a la última versión:

    composer global require laravel/valet
    

Una vez que se ha descargado Valet, se debe ejecutar el comando `install`:

    valet install
    valet restart
    

Tras actualizar, puede que sea necesario re-vincular sus sitios.

<a name="serving-sites"></a>

## Servir Sitios

Una vez que Valet se ha instalado, ya está listo para servir aplicaciones. Valet provee dos comandos para este propósito: `park` y `link`.

<a name="the-park-command"></a>
**El Comando `park`**

<div class="content-list">
  <ul>
    <li>
      Cree un nuevo directorio en su Mac como por ejemplo <code>mkdir ~/Sites</code>. A continuación, <code>cd ~/Sites</code> y ejecute <code>valet park</code>. Este comando incluirá el directorio actual en la lista que Valet debe buscar por sitios.
    </li>
    <li>
      A continuación, se puede crear una aplicación Laravel en este directorio <code>laravel new blog</code>.
    </li>
    <li>
      Acceder a <code>http://blog.dev</code> en el navegador.
    </li>
  </ul>
</div>

**Eso es todo.** Cualquier proyecto que se cree en el directorio "*parked*" se servirá automáticamente utilizando la convención `http://folder-name.dev`.

<a name="the-link-command"></a>
**El Comando `link`**

El comando `link` se puede utilizar para servir sitios Laravel. Este comando resulta útil si se pretende servir un sitio en un directorio concreto pero no el directorio completo.

<div class="content-list">
  <ul>
    <li>
      Para utilizar el comando, hay que navegar hasta el directorio y ejecutar <code>valet link nombre</code> en la terminal. Valet creará un enlace simbólico en <code>~/.valet/Sites</code> que apuntará al directorio de la aplicación.
    </li>
    <li>
      Tras ejecutar el comando <code>link</code>, se puede acceder al sitio desde el navegador <code>http://nombre.dev</code>.
    </li>
  </ul>
</div>

Para ver la lista de todos los directorios enlazados, se puede ejecutar el comando `valet links`. Se puede además utilizar el comando `valet unlink nombre` para eliminar un enlace simbólico.

> {tip} Es posible utilizar `valet link` para servir el mismo proyecto desde varios (sub)dominios. Para añadir un subdominio u otro dominio al proyecto, simplemente ejecutar `valet link subdominio.nombre` desde el directorio del proyecto.

<a name="securing-sites"></a>
**Asegurando Sitios con TLS**

Por defecto, Valet sirve los sitios a través de HTTP. Sin embargo, si se pretende servir un sitio con encriptación TLS utilizando HTTP/2, utilizar el comando `secure`. Por ejemplo, si el sitio se sirve bajo el dominio `laraveles.dev`:

    valet secure laravel
    

Para "insegurizar" un sitio y volver a servir el tráfico sobre HTTP, utilizar el comando `unsecure`. Así como el comando `secure`, acepta el nombre de *host* al que se aplica:

    valet unsecure laravel
    

<a name="sharing-sites"></a>

## Compartir Sitios

Valet incluye además un comando para compartir los sitios locales con el mundo. No precisa de software adicional una vez que Valet está instalado.

Para compartir un sitio, simplemente hay que navegar al directorio en la terminal y ejecutar el comando `valet share`. Se generará una URL de acceso público y quedará automáticamente copiada al portapapeles lista para ser pegada en el navegador. Eso es todo.

Para dejar de compartir un sitio, `Control + C` cancelará el proceso.

> {note} `valet share` no soporta el compartir sitios que han sido asegurados utilizando el comando `valet secure`.

<a name="custom-valet-drivers"></a>

## Drivers de Valet Personalizados

Se pueden desarrollar "drivers" para Valet propios para servir aplicaciones PHP que ejecutan otros frameworks o CMS que no se soporten de forma nativa. Al instalar Valet, se crea un directorio `~/.valet/Drivers` que contiene un archivo `SampleValetDriver.php`. Este archivo contiene un ejemplo de una implementación para demostrar como sería un driver personalizado. Un driver personalizado requiere tres métodos: `serves`, `isStaticFile` y `frontControllerPath`.

Todos estos métodos reciben `$sitePath`, `$siteName` y `$uri` como argumentos. `$sitePath` es el directorio completo del sitio que se está sirviendo en la máquina, por ejemplo `/Users/Laraveles/Sites/my-project`. `$siteName` es el la porción del dominio "host" / "nombre del sitio" (`my-project`). `$uri` es la URI entrante (`/foo/bar`).

Una vez que se ha programado el driver, se almacenará en `~/.valet/Drivers` utilizando la convención de nombres `FrameworkValetDriver.php`. Por ejemplo, si se está desarrollando un driver para WordPress, el nombre del archivo debería ser `WordPressValetDriver.php`.

A continuación se muestra un ejemplo de cada método que el driver personalizado debe implementar.

#### El Método `serves`

El método `serves` retornará `true` si el driver debe ser responsable de la petición entrante. De otro modo, retornará `false`. Por lo tanto, en este método se debe intentar determinar si `$sitePath` contiene un proyecto del tipo que se está intentando servir.

Por ejemplo, imaginemos que se está programando ` WordPressValetDriver`. El método debería ser algo así:

    /**
     * Determine if the driver serves the request.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return bool
     */
    public function serves($sitePath, $siteName, $uri)
    {
        return is_dir($sitePath.'/wp-admin');
    }
    

#### El Método `isStaticFile`

El método `isStaticFile` determinará si la petición entrante busca un archivo estático como una imagen u hoja de estilos. Si el archivo es estático, se retornará la ruta completa del archivo en el disco. Si no es un archivo estático, este método debe retornar `false`:

    /**
     * Determine if the incoming request is for a static file.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string|false
     */
    public function isStaticFile($sitePath, $siteName, $uri)
    {
        if (file_exists($staticFilePath = $sitePath.'/public/'.$uri)) {
            return $staticFilePath;
        }
    
        return false;
    }
    

> {note} El método `isStaticFile` se llamará únicamente si el método `serves` retorna `true` para la petición entrante y si la URI no es `/`.

#### El Método `frontControllerPath`

El método `frontControllerPath` retornará la ruta completa del controlador principal de la aplicación, normalmente un archivo "index.php" o equivalente:

    /**
     * Get the fully resolved path to the application's front controller.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string
     */
    public function frontControllerPath($sitePath, $siteName, $uri)
    {
        return $sitePath.'/public/index.php';
    }
    

<a name="local-drivers"></a>

### Drivers Locales

Para definir un driver de Valet personalizado para una aplicación concreta, hay que crear un `LocalValetDriver.php` en el directorio raíz de la aplicación. El driver puede heredar de la clase `ValetDriver` o cualquier otro específico como `LaravelValetDriver`:

    class LocalValetDriver extends LaravelValetDriver
    {
        /**
         * Determine if the driver serves the request.
         *
         * @param  string  $sitePath
         * @param  string  $siteName
         * @param  string  $uri
         * @return bool
         */
        public function serves($sitePath, $siteName, $uri)
        {
            return true;
        }
    
        /**
         * Get the fully resolved path to the application's front controller.
         *
         * @param  string  $sitePath
         * @param  string  $siteName
         * @param  string  $uri
         * @return string
         */
        public function frontControllerPath($sitePath, $siteName, $uri)
        {
            return $sitePath.'/public_html/index.php';
        }
    }
    

<a name="other-valet-commands"></a>

## Otros Comandos de Valet

| Comando           | Descripción                                                                                               |
| ----------------- | --------------------------------------------------------------------------------------------------------- |
| `valet forget`    | Ejecutar este comando desde un directorio "parked" (aparcado) para eliminarlo de la lista de directorios. |
| `valet paths`     | Ver todos los directorios aparcados.                                                                      |
| `valet restart`   | Reiniciar el *daemon* de Valet.                                                                           |
| `valet start`     | Iniciar el *daemon* de Valet.                                                                             |
| `valet stop`      | Detener el *daemon* de Valet.                                                                             |
| `valet uninstall` | Desinstalar completamente el *daemon* de Valet.                                                           |