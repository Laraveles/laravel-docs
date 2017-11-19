# Instalación

- [Instalación](#installation) 
    - [Requisitos del servidor](#server-requirements)
    - [Instalar Laravel](#installing-laravel)
    - [Configuración](#configuration)
- [Configuración del servidor Web](#web-server-configuration) 
    - [URL amigables](#pretty-urls)

<a name="installation"></a>

## Instalación

> {video} ¿Prefieres aprender con vídeos? Laracasts ofrece una [introducción gratuita y completa a Laravel](http://laravelfromscratch.com) para los recién llegados al *framework*. Es un buen lugar para comenzar tu viaje.

<a name="server-requirements"></a>

### Requisitos del servidor

El *framework* Laravel tiene unos pocos requerimientos de sistema. Por supuesto estos requisitos son satisfechos por la máquina virtual *[Laravel Homestead](/docs/{{version}}/homestead)*, por lo que es muy recomendable utilizar *Homestead* como su entorno de desarrollo local para Laravel.

No obstante si no está utilizando Homestead, necesitará asegurarse que el servidor cumple con los siguientes requermientos:

<div class="content-list">
  <ul>
    <li>
      PHP >= 7.0.0
    </li>
    <li>
      OpenSSL PHP Extension
    </li>
    <li>
      PDO PHP Extension
    </li>
    <li>
      Mbstring PHP Extension
    </li>
    <li>
      Tokenizer PHP Extension
    </li>
    <li>
      XML PHP Extension
    </li>
  </ul>
</div>

<a name="installing-laravel"></a>

### Instalar Laravel

Laravel utiliza [Composer](https://getcomposer.org) para gestionar sus dependencias. Por lo tanto, antes de utilizar Laravel, asegurese de tener instalado Composer en su máquina.

#### A través del Instalador de Laravel

En primer lugar, descargar al instalador de Laravel usando composer:

    composer global require "laravel/installer"
    

Asegurese de colocar el directorio del proveedor en su `$PATH` para hacerlo accesible a todo el sistema de ficheros para que el ejecutable Laravel pueda ser localizado. Este directorio existe en diferentes sitios dependiendo de su sistema operativo; no obstante, algunas localizaciones comunes incluyen:

<div class="content-list">
  <ul>
    <li>
      MacOS: <code>$HOME/.composer/vendor/bin</code>
    </li>
    <li>
      GNU / Linux Distributions: <code>$HOME/.config/composer/vendor/bin</code>
    </li>
  </ul>
</div>

Una vez instalado, el comando `laravel new` creará una instalación nueva de Laravel en el directorio que se especifique. Por ejemplo, `laravel new blog` creará un directorio llamado `blog` conteniendo éste una instalación limpia de Laravel con todas las dependencias instaladas:

    laravel new blog
    

#### Vía *composer create-project*

También se puede instalar Laravel ejecutando el comando de Composer `create-project` en la terminal:

    composer create-project --prefer-dist laravel/laravel blog
    

#### Servidor de desarrollo local

Si se tiene PHP instalado localmente y desea utilizar el servidor de desarrollo *built-in* (incluido) en PHP para su aplicación, puede utilizar el comando Artisan `serve`. Este comando arrancará un servidor de desarrollo accesible en `http://localhost:8000`:

    php artisan serve
    

Por supuesto, las opciones más robustas para desarrollo local son [Homestead](/docs/{{version}}/homestead) y [Valet](/docs/{{version}}/valet).

<a name="configuration"></a>

### Configuración

#### Directorio *public*

Después de instalar Laravel, debería configurar la raíz de su servidor web para que apunte al directorio `public`. El archivo `index.php` en este directorio sirve como *front controller* para todas las peticiones HTTP que entren en su aplicación.

#### Ficheros de configuración

Todos los archivos de configuración de Laravel Framework se encuentran en el directorio `config`. Cada opción está documentada, por lo que es más que recomendable navegar entre los diferentes archivos y conocer las diferentes opciones.

#### Permisos de directorios

Después de instalar Laravel, puede ser necesario configurar algunos permisos. Los directorios dentro de `storage` y de `bootstrap/cache` deberían tener permisos de escritura para el usuario del servidor web o Laravel no funcionará. Si se utiliza la máquina virtual [Homestead](/docs/{{version}}/homestead), estos permisos ya deben estar configurados.

#### Clave de la aplicación

Lo siguiente que se debe hacer una vez instalado Laravel es establecer la clave de aplicación a una cadena aleatoria. Si se instala Laravel utilizando Composer o el instalador Laravel, esta clave se habrá generado automáticamente a través del comando `php artisan key:generate`.

Normalmente, esta cadena debe contener 32 caracteres de longitud. Esta clave se debe establecer en el archivo de entorno `.env`. Si no se ha renombrado el archivo `.env.example` a `.env`, debe hacerse ahora. **¡Si no se establece la clave de aplicación, las sesiones de usuario y otros datos codificados no serán seguros!**

#### Configuración adicional

Laravel no necesita casi ninguna otra configuración para comenzar. ¡Ya puede empezar a programar! Sin embargo, puede querer revisar el archivo `config/app.php` y su documentación. Contiene varias opciones como `timezone` y `locale` que podrías desear cambiar en función de su aplicación.

También se pueden configurar algunos componentes adicionales de Laravel, tales como:

<div class="content-list">
  <ul>
    <li>
      <a href="/docs/{{version}}/cache#configuration">Cache</a>
    </li>
    <li>
      <a href="/docs/{{version}}/database#configuration">Database</a>
    </li>
    <li>
      <a href="/docs/{{version}}/session#configuration">Session</a>
    </li>
  </ul>
</div>

<a name="web-server-configuration"></a>

## Configuración del servidor Web

<a name="pretty-urls"></a>

### URL amigables

#### Apache

Laravel incluye un fichero `public/.htaccess` que es utilizado para proveer URLs sin el *front controller* `index.php` en la ruta. Antes de servir Laravel con Apache, asegurese de activar el módulo `mod_rewrite` para que el servidor respete el fichero `.htaccess`.

Si el fichero `.htaccess` que provee Laravel no funciona con su instalación de Apache, intente está alternativa:

    Options +FollowSymLinks
    RewriteEngine On
    
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]
    

#### Nginx

Si está utilizando Nginx, la siguiente directiva en la configuración de su sitio redireccionará todas las peticiones al controlador frontal `index.php`:

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    

Por supuesto, cuando se utiliza [Homestead](/docs/{{version}}/homestead) o [Valet](/docs/{{version}}/valet), las URLs serán automáticamente configuradas.