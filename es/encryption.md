# Encriptación

- [Introducción](#introduction)
- [Configuración](#configuration)
- [Cómo usar el encriptador](#using-the-encrypter)

<a name="introduction"></a>

## Introducción

El encriptador de Laravel utiliza OpenSSL para proporcionar encriptación AES-256 y AES-128. Se le recomienda encarecidamente que utilice las funciones de encriptación incorporadas de Laravel y que no intente utilizar sus propios algoritmos de encriptación "caseros". Todos los valores encriptados de Laravel son firmados usando un código de autenticación de mensaje (MAC) para que su valor subyacente no pueda ser modificado una vez encriptado.

<a name="configuration"></a>

## Configuración

Antes de usar el encriptador de Laravel, debe establecer una opción `key` en su archivo de configuración `config/app.php`. Debe usar el comando Artisan `php artisan key:generate` para generar esta clave ya que este comando usará el generador seguro de bytes aleatorios de PHP para construir su clave. If this value is not properly set, all values encrypted by Laravel will be insecure.

<a name="using-the-encrypter"></a>

## Using The Encrypter

#### Encrypting A Value

You may encrypt a value using the `encrypt` helper. All encrypted values are encrypted using OpenSSL and the `AES-256-CBC` cipher. Furthermore, all encrypted values are signed with a message authentication code (MAC) to detect any modifications to the encrypted string:

    <?php
    
    namespace App\Http\Controllers;
    
    use App\User;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;
    
    class UserController extends Controller
    {
        /**
         * Store a secret message for the user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function storeSecret(Request $request, $id)
        {
            $user = User::findOrFail($id);
    
            $user->fill([
                'secret' => encrypt($request->secret)
            ])->save();
        }
    }
    

#### Encrypting Without Serialization

Encrypted values are passed through `serialize` during encryption, which allows for encryption of objects and arrays. Thus, non-PHP clients receiving encrypted values will need to `unserialize` the data. If you would like to encrypt and decrypt values without serialization, you may use the `encryptString` and `decryptString` methods of the `Crypt` facade:

    use Illuminate\Support\Facades\Crypt;
    
    $encrypted = Crypt::encryptString('Hello world.');
    
    $decrypted = Crypt::decryptString($encrypted);
    

#### Decrypting A Value

You may decrypt values using the `decrypt` helper. If the value can not be properly decrypted, such as when the MAC is invalid, an `Illuminate\Contracts\Encryption\DecryptException` will be thrown:

    use Illuminate\Contracts\Encryption\DecryptException;
    
    try {
        $decrypted = decrypt($encryptedValue);
    } catch (DecryptException $e) {
        //
    }