# Almacenamiento de archivos – *File storage*

- [Introducción](#introduction)
- [Configuración](#configuration) 
    - [El disco *public*](#the-public-disk)
    - [El *driver* local](#the-local-driver)
    - [Prerrequisitos del *driver*](#driver-prerequisites)
- [Obtener Instancias de discos](#obtaining-disk-instances)
- [Obtener archivos](#retrieving-files) 
    - [URLs de archivos](#file-urls)
    - [Metadatos de archivos](#file-metadata)
- [Almacenar archivos](#storing-files) 
    - [Subida de archivos](#file-uploads)
    - [Visibilidad de archivos](#file-visibility)
- [Eliminar archivos](#deleting-files)
- [Directorios](#directories)
- [*Filesystems* personalizados](#custom-filesystems)

<a name="introduction"></a>

## Introducción

Laravel provee una potente abstracción para *filesystem* (manejo de archivos) gracias al paquete de PHP de Frank de Jonge [Flysystem](https://github.com/thephpleague/flysystem). La integración de Flysystem de Laravel provee *drivers* sencillos para trabajar con sistemas de archivos locales, Amazon S3, y Rackspace Cloud. Incluso mejor, es increíblemente sencillo alternar entre estas opciones de almacenamiento, pues el API se mantiene constante para cada sistema.

<a name="configuration"></a>

## Configuración

La configuración de *filesystem* se encuentra en el fichero `config/filesystems.php`. En este archivo se pueden configurar todos los "discos". Cada disco representa un *driver* de almacenamiento y una ubicación en particular. En el archivo de configuración se incluyen varios ejemplos para cada *driver* soportado. Así pues, simplemente hay que modificar la configuración para reflejar las preferencias de almacenamiento y credenciales.

Por supuesto, se pueden configurar tantos discos como sea necesario, e incluso varios discos para un mismo *driver*.

<a name="the-public-disk"></a>

### El disco *public*

El disco `public` está previsto para archivos que van a ser de acceso público. Por defecto, el uso del disco `public` usa el *driver* `local` y almacena estos archivos en `storage/app/public`. Para acceder desde la web, debe crear un enlace simbólico desde `public/storage` a `Storage/app/public`. Esta convención mantendrá sus archivos de acceso público en un directorio que se puede compartir fácilmente en las implementaciones cuando se utilizan sistemas de implementación sin tiempo de inactividad como [Envoyer](https://envoyer.io).

Para crear el link simbólico, puede usar el comando Artisan `storage:link`:

    php artisan storage:link
    

Por supuesto, una vez que se ha almacenado un archivo y se ha creado el link simbólico, puede crear una URL para los archivos utilizando el *helper* `asset`:

    echo asset('storage/file.txt');
    

<a name="the-local-driver"></a>

### El *driver* local

Cuando se utiliza el *driver* `local`, hay que tener en cuenta que todas las operaciones son relativas al directorio `root` definido en el archivo de configuración. Por defecto, este valor se establece en el directorio `storage/app`. Por lo tanto, el siguiente comando almacenará un archivo en `storage/app/file.txt`:

    Storage::disk('local')->put('file.txt', 'Contents');
    

<a name="driver-prerequisites"></a>

### Pre requisitos del *driver*

#### Paquetes composer

Antes de utilizar los *drivers* S3 o Rackspace, es necesario instalar el paquete apropiado vía Composer:

- Amazon S3: `league/flysystem-aws-s3-v3 ~1.0`
- Rackspace: `league/flysystem-rackspace ~1.0`

#### Configuración del *driver* S3

La información de configuración del *driver* S3 se encuentra localizada en el archivo de configuración `config/database.php`. Este archivo contiene un ejemplo de *array* de configuración para el *driver* S3. Podrá modificar este *array* con su propia configuración y credenciales de S3. Por conveniencia, estas variables de entorno coinciden con la convención de nomenclatura utilizada por AWS CLI.

#### Configuración FTP

Las integraciones de Laravel en Flysystem funcionan muy bien con FTP; sin embargo, no se incluye ninguna configuración de ejemplo en el archivo de configuración `filesystems.php`. Si necesita configurar el *filesystem* FTP, puede usar el siguiente ejemplo de configuración:

    'ftp' => [
        'driver'   => 'ftp',
        'host'     => 'ftp.example.com',
        'username' => 'your-username',
        'password' => 'your-password',
    
        // Optional FTP Settings...
        // 'port'     => 21,
        // 'root'     => '',
        // 'passive'  => true,
        // 'ssl'      => true,
        // 'timeout'  => 30,
    ],
    

#### Configuración del *driver* Rackspace

Las integraciones Flysystem de Laravel funcionan muy bien con Rackspace; sin embargo, no se incluye una configuración de ejemplo en el archivo de configuración `filesystems.php</ 0> del framework. Si necesita configurar el <em>filesystem</em> Rackspace, puede usar el siguiente ejemplo de configuración:</p>

<pre><code>'rackspace' => [
    'driver'    => 'rackspace',
    'username'  => 'your-username',
    'key'       => 'your-key',
    'container' => 'your-container',
    'endpoint'  => 'https://identity.api.rackspacecloud.com/v2.0/',
    'region'    => 'IAD',
    'url_type'  => 'publicURL',
],
`</pre> 

<a name="obtaining-disk-instances"></a>

## Obtener Instancias de discos

La *facade* `Storage` se puede utilizar para interactuar con cualquiera de los discos configurados. Por ejemplo, se puede utilizar el método `put` de la *facade* para almacenar un avatar en el disco por defecto. Si se llaman a métodos en la *facade* `Storage` sin haber llamado primero al método `disk`, la llamada al método se pasará automáticamente al disco por defecto:

    use Illuminate\Support\Facades\Storage;
    
    Storage::put('avatars/1', $fileContents);
    

Si la aplicación interacciona con varios discos, se puede utilizar el método `disk` en la *facade* `Storage` para trabajar con los archivos de un disco en particular:

    Storage::disk('s3')->put('avatars/1', $fileContents);
    

<a name="retrieving-files"></a>

## Obtener archivos

El método `get` se utiliza para obtener el contenido de un archivo. El método devolverá una cadena sin formato con el contenido del archivo. Recordar, todas las rutas a archivos deben especificarse de forma relativa a la "raíz" configurada para el disco:

    $contents = Storage::get('file.jpg');
    

El método `exists` puede utilizarse para determinar si existe un archivo en el disco:

    $exists = Storage::disk('s3')->exists('file.jpg');
    

<a name="file-urls"></a>

### URLs de archivos

Puede usar el método `url` para obtener la URL para un archivo dado. Si está usando el *driver* `local`, normalmente antepondrá `/storage` a la ruta y retornará la URL relativa del archivo. Si está usando el *driver* `s3` ó `rackspace`, se retornará la URL remota completa:

    use Illuminate\Support\Facades\Storage;
    
    $url = Storage::url('file1.jpg');
    

> {nota} Recuerde, si está usando el *driver* `local`, todos los archivos que deberían ser de acceso público deben colocarse en el directorio `storage/app/public</ 0>. Además, deberá <a href="#the-public-disk">crear un enlace simbólico</a> en <code>public/storage` que apuntará al directorio `storage/app/public`.

#### URLs temporales

Para archivos almacenados usando los *driver* `s3` o `rackspace`, puede crearse una URL temporal para un archivo dado usando el método `temporaryURL`. Este método acepta una ruta e instancia `DateTime` especificando cuando debería expirar la URL:

    $url = Storage::temporaryUrl(
        'file1.jpg', Carbon::now()->addMinutes(5)
    );
    

#### Personalizar la URL del servidor local

Si quiere predefinir el *host* para los archivos almacenados en un disco usando el *driver* `local`, puede añadir la opción `url` al *array* de configuración del disco:

    'public' => [
        'driver' => 'local',
        'root' => storage_path('app/public'),
        'url' => env('APP_URL').'/storage',
        'visibility' => 'public',
    ],
    

<a name="file-metadata"></a>

### Metadatos de los archivos

Además de leer y escribir archivos, Laravel permite obtener información sobre los propios archivos. Por ejemplo, el método `size` puede utilizarse para obtener el tamaño del archivo en bytes:

    use Illuminate\Support\Facades\Storage;
    
    $size = Storage::size('file1.jpg');
    

El método `lastModified` retorna el UNIX timestamp de la última vez que el archivo fue modificado:

    $time = Storage::lastModified('file1.jpg');
    

<a name="storing-files"></a>

## Almacenar archivos

El método `put` puede usarse para almacenar el contenido de archivos raw en un disco. Además se puede pasar un `recurso` de PHP al método `put`, el cual utilizará el soporte para *stream* subyacente en Flysystem. Es recomendable utilizar *streams* cuando se trabaja con archivos grandes:

    use Illuminate\Support\Facades\Storage;
    
    Storage::put('file.jpg', $contents);
    
    Storage::put('file.jpg', $resource);
    

#### *Streaming* automático

Si desea que Laravel administre automáticamente el *streaming* de un archivo concreto en una ubicación de almacenamiento, puede utilizar el método `putFile` o el método `putFileAs`. Este método acepta tanto una instancia `Iluminate\Http\File` como de `Illuminate\Http\UploadedFile` y automáticamente enviará el archivo a la ubicación deseada:

    use Illuminate\Http\File;
    use Illuminate\Support\Facades\Storage;
    
    // Automatically generate a unique ID for file name...
    Storage::putFile('photos', new File('/path/to/photo'));
    
    // Manually specify a file name...
    Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');
    

Hay varias cuestiones importantes a tener en cuenta sobre el método `putFile`: Tenga en cuenta que únicamente hay que especificar el nombre de un directorio, no el nombre de un archivo. Por defecto, el método `putFile` generará un ID único equivalente al nombre del archivo. El métoto `putFile` devolverá la ruta del archivo, incluyendo el nombre del archivo, por lo que podría utilizarla para almacenarla en su base de datos.

Los métodos `putFile` y `putFileAs` también aceptan un argumento para especificar la "visibilidad" del archivo almacenado. Esto es particularmente útil si se está almacenando el archivo en un disco en la nube como S3 y se necesita que dicho archivo sea accesible públicamente:

    Storage::putFile('photos', new File('/path/to/photo'), 'public');
    

#### Añadir datos al principio o al final de archivos

Los métodos `prepend` y `append` permiten escribir al principio o al final de un archivo:

    Storage::prepend('file.log', 'Prepended Text');
    
    Storage::append('file.log', 'Appended Text');
    

#### Copiar & mover archivos

El método `copy` puede usarse para copiar un archivo existente en una nueva ubicación del disco, mientras que el método `move` puede utilizarse para renombrar o mover un archivo existente a una nueva ubicación:

    Storage::copy('old/file1.jpg', 'new/file1.jpg');
    
    Storage::move('old/file1.jpg', 'new/file1.jpg');
    

<a name="file-uploads"></a>

### Subida de archivos

En las aplicaciones web, uno de los casos de uso más comunes de almacenamiento de archivos es almacenar los archivos subidos por los usuarios, tales como imágenes, fotos y documentos. Laravel permite almacenar los archivos subidos fácilmente usando el método `store` para una instancia de archivo subido. Simplemente hay que llamar al método `store` con la trayectoria en la que deseas almacenar el archivo subido:

    <?php
    
    namespace App\Http\Controllers;
    
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;
    
    class UserAvatarController extends Controller
    {
        /**
         * Update the avatar for the user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            $path = $request->file('avatar')->store('avatars');
    
            return $path;
        }
    }
    

Hay varias cuestiones importantes a tener en cuenta en este ejemplo. Nótese que sólo se ha especificado un nombre de directorio, no un nombre de archivo. Por defecto, el método `store` generará un ID único equivalente al nombre del archivo. El método `store` devolverá la ruta del archivo, y así podrá utilizarse esta, incluyendo el nombre de archivo generado, para almacenarla en su base de datos.

También puede llamar al método `putFile` de la *facade* `Storage` para realizar la misma manipulación del archivo como en el siguiente ejemplo:

    $path = Storage::putFile('avatars', $request->file('avatar'));
    

#### Especificar un nombre de archivo

Si no quiere asignar un nombre automáticamente al archivo almacenado, puede utilizar el método `storeAs`, el cual recibe la ruta, el nombre del archivo y (opcionalmente) el disco como argumentos:

    $path = $request->file('avatar')->storeAs(
        'avatars', $request->user()->id
    );
    

Por supuesto, puede usar el método `putFileAs` de la *facade* `Storage`, el cual permitirá la misma manipulación de archivos como en el ejemplo anterior:

    $path = Storage::putFileAs(
        'avatars', $request->file('avatar'), $request->user()->id
    );
    

#### Especificar un disco

Por defecto, este método usará el disco predeterminado. Si quiere especificar otro disco, debe pasar el nombre del disco como segundo argumento en el método `store`:

    $path = $request->file('avatar')->store(
        'avatars/'.$request->user()->id, 's3'
    );
    

<a name="file-visibility"></a>

### Visibilidad de archivos

En la integración de Flysystem en Laravel, "visibilidad" es una abstracción de los permisos de archivo para múltiples plataformas. Los archivos pueden ser declarados `public` o `private`. Cuando un archivo es declarado `public`, se está indicando que el archivo debería ser accesible para otros. Por ejemplo, cuando se usa el *driver* de S3, puede obtener las URLs de los archivos `public`.

Puede seleccionar la visibilidad cuando configura el archivo a través del método `put`:

    use Illuminate\Support\Facades\Storage;
    
    Storage::put('file.jpg', $contents, 'public');
    

Si el archivo ya ha sido almacenado, su visibilidad puede obtenerse o seleccionarse a través de los métodos `getVisibility` y `setVisibility` respectivamente:

    $visibility = Storage::getVisibility('file.jpg');
    
    Storage::setVisibility('file.jpg', 'public')
    

<a name="deleting-files"></a>

## Eliminar archivos

El método `delete` acepta un archivo o un *array* de archivos a eliminar del disco:

    use Illuminate\Support\Facades\Storage;
    
    Storage::delete('file.jpg');
    
    Storage::delete(['file1.jpg', 'file2.jpg']);
    

Si es necesario, se puede especificar el disco en el que se debería eliminar el archivo:

    use Illuminate\Support\Facades\Storage;
    
    Storage::disk('s3')->delete('folder_path/file_name.jpg');
    

<a name="directories"></a>

## Directorios

#### Obtener todos los archivos de un directorio

El método `files` retorna un *array* de todos los archivos en un directorio concreto. Para recuperar una lista de todos los archivos de un directorio, incluyendo sus sub-directorios, se puede utilizar el método `allFiles`:

    use Illuminate\Support\Facades\Storage;
    
    $files = Storage::files($directory);
    
    $files = Storage::allFiles($directory);
    

#### Obtener todos los directorios de un directorio

El método `directories` retorna un *array* de todos los directorios dentro de un directorio. Además, se puede utilizar el método `allDirectories` para obtener una lista de todos los directorios de un directorio y sus sub-directorios:

    $directories = Storage::directories($directory);
    
    // Recursive...
    $directories = Storage::allDirectories($directory);
    

#### Crear un directorio

El método `makeDirectory` creará el directorio, incluyendo cualquier sub-directorio que se necesite:

    Storage::makeDirectory($directory);
    

#### Eliminar un directorio

Finalmente, se puede usar `deleteDirectory` para eliminar un directorio y todos sus archivos:

    Storage::deleteDirectory($directory);
    

<a name="custom-filesystems"></a>

## *Filesystems* personalizados

La integración de Flysystem de Laravel provee *drivers* para varios "*drivers*"; sin embargo, Flysystem no está limitado a estos y posee adaptadores para otros sistemas de almacenamiento. Se puede crear un *driver* personalizado si se necesita alguno de estos adaptadores adicionales en una aplicación Laravel.

Para configurar un sistema de archivos a medida se necesitará un adaptador Flysystem. Se puede añadir un adaptador de Dropbox mantenido por la comunidad a un proyecto utilizando:

    composer require spatie/flysystem-dropbox
    

Puede crear un [service provider](/docs/{{version}}/providers) llamado `DropboxServiceProvider`. En el método `boot` del *provider*, se puede utilizar el método `extend` de la *facade* `Storage` para definir el *driver*:

    <?php
    
    namespace App\Providers;
    
    use Storage;
    use League\Flysystem\Filesystem;
    use Illuminate\Support\ServiceProvider;
    use Spatie\Dropbox\Client as DropboxClient;
    use Spatie\FlysystemDropbox\DropboxAdapter;
    
    class DropboxServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Storage::extend('dropbox', function ($app, $config) {
                $client = new DropboxClient(
                    $config['authorizationToken']
                );
    
                return new Filesystem(new DropboxAdapter($client));
            });
        }
    
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }
    

El primer parámetro del método `extend` es el nombre del *driver* y el segundo un *Closure* que recibirá las variables `$app` y `$config`. El resultado del *Closure* debe retornar una instancia de `League\Flysystem\Filesystem`. La variable `$config` contiene los valores definidos en `config/filesystems.php` para el disco especificado.

Una vez creado el *service provider*, para registrar la extensión se puede utilizar el *driver* `dropbox` en el archivo de configuración `config/filesystems.php`.