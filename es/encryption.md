# Encriptación

- [Introducción](#introduction)
- [Configuración](#configuration)
- [Cómo usar el encriptador](#using-the-encrypter)

<a name="introduction"></a>

## Introducción

El encriptador de Laravel utiliza OpenSSL para proporcionar encriptación AES-256 y AES-128. Se le recomienda encarecidamente que utilice las funciones de encriptación incorporadas de Laravel y que no intente utilizar sus propios algoritmos de encriptación "caseros". Todos los valores encriptados de Laravel son firmados usando un código de autenticación de mensaje (MAC) para que su valor subyacente no pueda ser modificado una vez encriptado.

<a name="configuration"></a>

## Configuración

Antes de usar el encriptador de Laravel, debe establecer una opción `key` en su archivo de configuración `config/app.php`. Debe usar el comando Artisan `php artisan key:generate` para generar esta clave ya que este comando usará el generador seguro de bytes aleatorios de PHP para construir su clave. Si este valor no se establece correctamente, los valores cifrados por Laravel serán inseguros.

<a name="using-the-encrypter"></a>

## Cómo usar el encriptador

#### Cifrar un valor

Puede encriptar un valor usando el *helper* `encrypt`. Todos los valores se encriptan utilizando OpenSSL y el algoritmo de cifrado `AES-256-CBC`. Además, todos los valores cifrados se firman con un código de autenticación de mensaje (MAC) para detectar cualquier modificación de la cadena cifrada:

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
    

#### Encriptación sin serialización

Los valores codificados se transmiten a través de `serialize` durante el encriptado, lo que permite la encriptación de objetos y matrices. Por lo tanto, los clientes no PHP que reciban valores cifrados necesitarán usar `unserialize` para los datos. Si desea encriptar y desencriptar valores sin serialización, puede utilizar los métodos `encryptString` y `decryptString` de la *facade* `Crypt`:

    use Illuminate\Support\Facades\Crypt;
    
    $encrypted = Crypt::encryptString('Hello world.');
    
    $decrypted = Crypt::decryptString($encrypted);
    

#### Desencriptar un valor

Puede desencriptar valores utilizando el *helper* `decrypt`. Si no se puede desencriptar correctamente, como cuando la MAC es inválida, se lanzará una `Illuminate\Contracts\Encryption\DecryptException`:

    use Illuminate\Contracts\Encryption\DecryptException;
    
    try {
        $decrypted = decrypt($encryptedValue);
    } catch (DecryptException $e) {
        //
    }