# Restablecer contraseñas

- [Introducción](#introduction)
- [Consideraciones de la base de datos](#resetting-database)
- [Rutas](#resetting-routing)
- [Vistas](#resetting-views)
- [Tras restablecer contraseñas](#after-resetting-passwords)
- [Personalizacion](#password-customization)

<a name="introduction"></a>

## Introducción

> {tip} **¿Quiere comenzar rápidamente?** Ejecute `php artisan make:auth` en una aplicación de Laravel recién instalada y visite `http://tu-app.dev/register` o cualquier otra URL que esté asignada a su aplicación. Este simple comando gestionará todo el *scaffolding* del sistema de autenticación, ¡incluso el restablecimiento de contraseñas!

La mayoría de las aplicaciones web proporcionan un medio para el restablecimiento de contraseñas olvidadas. En lugar de forzar la re-implementación en cada aplicación, Laravel provee de métodos para enviar recordatorios y restablecimiento de contraseñas.

> {note} Antes de usar la característica de Laravel de restablecimiento de contraseñas, su usuario debe utilizar el *trait* `Illuminate\Notifications\Notifiable`.

<a name="resetting-database"></a>

## Consideraciones de la Base de Datos

Para comenzar, verificar que el modelo `App\User` implementa el contrato `Illuminate\Contracts\Auth\CanResetPassword`. Por supuesto, el modelo `App\User` incluido con el framework ya implementa esta interfaz y utiliza el *trait* `Illuminate\Auth\Passwords\CanResetPassword` para incluir los métodos necesarios para cumplir la interfaz.

#### Generar la migración de la tabla de *tokens* para el restablecimiento

A continuación, se debe crear una tabla para almacenar los *tokens* de restablecimiento de contraseña. La migración de esta tabla se incluye con Laravel y se encuentra en el directorio `database/migrations`. Todo lo que hay que hacer es ejecutar las migraciones de la base de datos:

    php artisan migrate
    

<a name="resetting-routing"></a>

## Rutas

Laravel incluye las clases `Auth\ForgotPasswordController` y `Auth\ResetPasswordController` que contienen la lógica necesaria para enviar por correo los enlaces de restablecimiento de contraseñas y su propio restablecimiento. Todas las rutas necesarias para llevar a cabo la acción se pueden generar utilizando el comando de Artisan `make:auth`:

    php artisan make:auth
    

<a name="resetting-views"></a>

## Vistas

De nuevo, Laravel generará todas las vistas necesarias para el restablecimiento de contraseñas al ejecutar el comando `make:auth`. Las vistas se almacenarán en `resources/views/auth/passwords`. Se es libre de personalizarlas como se necesite en su aplicación.

<a name="after-resetting-passwords"></a>

## Tras restablecer contraseñas

Una vez que ha definido las rutas y vistas para restablecer las contraseñas de los usuarios, se puede acceder a través del navegador en `/password/reset`. El `ForgotPasswordController` que incluye el framework ya proporciona la lógica para enviar los e-mails de restablecimiento, mientras que `ResetPasswordController` incluye la lógica del restablecimiento en sí.

Después de que una contraseña se haya restablecido, el usuario será identificado en la aplicación y redirigido `/home`. Se puede personalizar la redirección tras el restablecimiento de contraseña definiendo la propiedad `redirectTo` en `ResetPasswordController`:

    protected $redirectTo = '/dashboard';
    

> {note} Por defecto, los *tokens* de restablecimiento de contraseña expiran en una hora. Se puede cambiar esto a través de la opción `expire` del archivo `config/auth.php`.

<a name="password-customization"></a>

## Personalizacion

#### Personalización del *Guard* de autenticación

En su archivo de configuración `auth.php` se pueden configurar varios "guards", que pueden utilizarse para definir el comportamiento de autenticación para varias tablas de usuarios. Se puede personalizar el `ResetPasswordController` incluido para utilizar el *guard* de su elección reemplazando el método `guard` en el controlador. Este método debe retornar una instancia de un *guard*:

    use Illuminate\Support\Facades\Auth;
    
    protected function guard()
    {
        return Auth::guard('guard-name');
    }
    

#### Personalizar el *password broker*

En el archivo de configuración `auth.php`, puede configurar varios "brokers" que se podrán utilizar para restablecer contraseñas en sistemas con varias tablas de usuarios. Se puede personalizar `ForgotPasswordController` y `ResetPasswordController` que se incluyen con Laravel para utilizar el *broker* de su elección sobrescribiendo el método `broker`:

    use Illuminate\Support\Facades\Password;
    
    /**
     * Get the broker to be used during password reset.
     *
     * @return PasswordBroker
     */
    protected function broker()
    {
        return Password::broker('name');
    }
    

#### Personalizar el e-mail de restablecimiento

Se puede modificar la clase de notificación a utilizar para enviar el enlace de restablecimiento de contraseña al usuario. Para comenzar, sobrescriba el método `sendPasswordResetNotification` en el modelo `User`. En este método puede enviar la notificación utilizando cualquier clase de notificación que desee. El `$token` de restablecimiento de contraseña es el primer parámetro que recibe el método:

    /**
     * Send the password reset notification.
     *
     * @param  string  $token
     * @return void
     */
    public function sendPasswordResetNotification($token)
    {
        $this->notify(new ResetPasswordNotification($token));
    }