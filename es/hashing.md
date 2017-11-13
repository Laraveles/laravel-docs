# Cifrado – *Hashing*

- [Introducción](#introduction)
- [Uso básico](#basic-usage)

<a name="introduction"></a>

## Introducción

La [facade](/docs/{{version}}/facades) `Hash` de Laravel proporciona un cifrado seguro para almacenar contraseñas de usuario. Si está utilizando las clases `LoginController` y `RegisterController` incorporadas en su aplicación Laravel, utilizarán automáticamente *Bcrypt* para el registro y la autenticación.

> {tip} *Bcrypt* es una gran opción para el cifrado de contraseñas ya que su "factor de trabajo" es ajustable, lo que significa que el tiempo necesario para generar un cifrado se puede aumentar a medida que aumenta la potencia del hardware.

<a name="basic-usage"></a>

## Uso básico

Se puede cifrar una contraseña llamando al método `make` de la *facade* `Hash`:

    <?php
    
    namespace App\Http\Controllers;
    
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use App\Http\Controllers\Controller;
    
    class UpdatePasswordController extends Controller
    {
        /**
         * Update the password for the user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // Validate the new password length...
    
            $request->user()->fill([
                'password' => Hash::make($request->newPassword)
            ])->save();
        }
    }
    

El método `make` también le permite administrar el factor de trabajo del algoritmo de cifrado de *bcrypt* usando la opción `rounds`; sin embargo, el valor predeterminado es aceptable para la mayoría de las aplicaciones:

    $hashed = Hash::make('password', [
        'rounds' => 12
    ]);
    

#### Verificar una contraseña contra un cifrado

El método `check` permite verificar que una cadena corresponde con un cifrado concreto. Sin embargo, si está usando `LoginController` [incluido con Laravel](/docs/{{version}}/authentication), probablemente no necesitará usar esto directamente, ya que este controlador llama automáticamente a este método:

    if (Hash::check('plain-text', $hashedPassword)) {
        // The passwords match...
    }
    

#### Comprobar si una contraseña require re-cifrado

La función `needsRehash` permite determinar si el factor de trabajo utilizado por el cifrado ha cambiado desde que se cifró la contraseña:

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }