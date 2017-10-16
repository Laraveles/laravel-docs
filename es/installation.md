# Instalación

- [Instalación](#installation) 
    - [Requisitos del Servidor](#server-requirements)
    - [Instalar Laravel](#installing-laravel)
    - [Configuración](#configuration)
- [Configuración del Servidor Web](#web-server-configuration) 
    - [URL Amigables](#pretty-urls)

<a name="installation"></a>

## Instalación

> ¿Prefieres aprender con vídeos {video}? Laracasts ofrece una [introducción gratuita y completa a Laravel](http://laravelfromscratch.com) para los recién llegados al *framework*. Es un buen lugar para comenzar tu viaje.

<a name="server-requirements"></a>

### Requisitos del Servidor

The Laravel framework has a few system requirements. Of course, all of these requirements are satisfied by the [Laravel Homestead](/docs/{{version}}/homestead) virtual machine, so it's highly recommended that you use Homestead as your local Laravel development environment.

However, if you are not using Homestead, you will need to make sure your server meets the following requirements:

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

### Installing Laravel

Laravel utilizes [Composer](https://getcomposer.org) to manage its dependencies. So, before using Laravel, make sure you have Composer installed on your machine.

#### Via Laravel Installer

First, download the Laravel installer using Composer:

    composer global require "laravel/installer"
    

Make sure to place the `$HOME/.composer/vendor/bin` directory (or the equivalent directory for your OS) in your $PATH so the `laravel` executable can be located by your system.

Once installed, the `laravel new` command will create a fresh Laravel installation in the directory you specify. For instance, `laravel new blog` will create a directory named `blog` containing a fresh Laravel installation with all of Laravel's dependencies already installed:

    laravel new blog
    

#### Via Composer Create-Project

Alternatively, you may also install Laravel by issuing the Composer `create-project` command in your terminal:

    composer create-project --prefer-dist laravel/laravel blog
    

#### Local Development Server

If you have PHP installed locally and you would like to use PHP's built-in development server to serve your application, you may use the `serve` Artisan command. This command will start a development server at `http://localhost:8000`:

    php artisan serve
    

Of course, more robust local development options are available via [Homestead](/docs/{{version}}/homestead) and [Valet](/docs/{{version}}/valet).

<a name="configuration"></a>

### Configuration

#### Public Directory

After installing Laravel, you should configure your web server's document / web root to be the `public` directory. The `index.php` in this directory serves as the front controller for all HTTP requests entering your application.

#### Configuration Files

All of the configuration files for the Laravel framework are stored in the `config` directory. Each option is documented, so feel free to look through the files and get familiar with the options available to you.

#### Directory Permissions

After installing Laravel, you may need to configure some permissions. Directories within the `storage` and the `bootstrap/cache` directories should be writable by your web server or Laravel will not run. If you are using the [Homestead](/docs/{{version}}/homestead) virtual machine, these permissions should already be set.

#### Application Key

The next thing you should do after installing Laravel is set your application key to a random string. If you installed Laravel via Composer or the Laravel installer, this key has already been set for you by the `php artisan key:generate` command.

Typically, this string should be 32 characters long. The key can be set in the `.env` environment file. If you have not renamed the `.env.example` file to `.env`, you should do that now. **If the application key is not set, your user sessions and other encrypted data will not be secure!**

#### Additional Configuration

Laravel needs almost no other configuration out of the box. You are free to get started developing! However, you may wish to review the `config/app.php` file and its documentation. It contains several options such as `timezone` and `locale` that you may wish to change according to your application.

You may also want to configure a few additional components of Laravel, such as:

<div class="content-list">
  <ul>
    <li>
      <a href="/docs/{{version}}/cache#configuration">Cache</a>
    </li>
    <li>
      <a href="/docs/{{version}}/database#configuration">Base de datos</a>
    </li>
    <li>
      <a href="/docs/{{version}}/session#configuration">Sesión</a>
    </li>
  </ul>
</div>

<a name="web-server-configuration"></a>

## Configuración del servidor Web

<a name="pretty-urls"></a>

### URL Amigables

#### Apache

Laravel incluye un fichero `public/.htaccess` que es utilizado para proveer URLs sin el *front controller* `index.php` en la ruta. Antes de servir Laravel con Apache, asegurese de activar el módulo `mod_rewrite` para que el servidor respete el fichero `.htaccess`.

Si el fichero `.htaccess` que provee Laravel no funciona con su instalación de Apache, intente está alternativa:

    Options +FollowSymLinks
    RewriteEngine On
    
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]
    

#### Nginx

Si usted está utilizando Nginx, la siguiente directiva en la configuración de su sitio redireccionará todas las peticiones al controlador frontal `index.php`:

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    

Por supuesto, cuando se utiliza [Homestead](/docs/{{version}}/homestead) o [Valet](/docs/{{version}}/valet), las URLs serán automáticamente configuradas.