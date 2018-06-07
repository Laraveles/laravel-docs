# Pruebas HTTP

- [Introducción](#introduction) 
    - [Personalización de los Request Headers](#customizing-request-headers)
- [Sesiones / Autenticación](#session-and-authentication)
- [Pruebas de los APIs JSON](#testing-json-apis)
- [Testeo de Subidas de Archivos](#testing-file-uploads)
- [Verificaciones disponibles](#available-assertions)

<a name="introduction"></a>

## Introducción

Laravel proporciona una API muy fluida para hacer peticiones HTTP a la aplicación y examinar la salida. Por ejemplo, echemos un vistazo a la prueba que se define a continuación:

    <?php
    
    namespace Tests\Feature;
    
    use Tests\TestCase;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    
    class ExampleTest extends TestCase
    {
        /**
         * A basic test example.
         *
         * @return void
         */
        public function testBasicTest()
        {
            $response = $this->get('/');
    
            $response->assertStatus(200);
        }
    }
    

El método `get` realiza una petición `GET` a la aplicación, mientras que el método `assertStatus` verifica que la respuesta devuelta debe tener el código de estado HTTP dado. Además de esta simple verificación, Laravel también contiene una variedad de verificaciones (asserts) para inspeccionar los encabezados de respuesta, contenido, estructura JSON y más.

<a name="customizing-request-headers"></a>

### Personalización de los Request Headers

Se puede usar el método `withHeader` para personalizar los encabezados de la solicitud antes de enviarla a la aplicación. Esto permite agregar los encabezados personalizados que desee a la solicitud:

    <?php
    
    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->withHeaders([
                'X-Header' => 'Value',
            ])->json('POST', '/user', ['name' => 'Sally']);
    
            $response
                ->assertStatus(200)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }
    

<a name="session-and-authentication"></a>

## Sesiones / Autenticación

Laravel proporciona varios ayudantes para trabajar con la sesión durante el testing HTTP. En primer lugar, se puede establecer los datos de la sesión en un array determinado utilizando el método `withSession`. Esto es útil para cargar la sesión con datos antes de emitir una solicitud a la aplicación:

    <?php
    
    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $response = $this->withSession(['foo' => 'bar'])
                             ->get('/');
        }
    }
    

Como es de esperar, uno de los usos más comunes para el uso de la sesión es la de mantener el estado de un usuario autenticado. El helper `actingAs` provee de una manera simple la posibilidad de autenticar a un usuario dado. Por ejemplo, se puede usar un [model factory](/docs/{{version}}/database-testing#writing-factories) para generar y autenticar a un usuario:

    <?php
    
    use App\User;
    
    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $user = factory(User::class)->create();
    
            $response = $this->actingAs($user)
                             ->withSession(['foo' => 'bar'])
                             ->get('/');
        }
    }
    

También se puede especificar qué *guard* se debe usar para autenticar al usuario pasando el nombre del mismo como segundo argumento del método `actingAs`:

    $this->actingAs($user, 'api')
    

<a name="testing-json-apis"></a>

## Pruebas de los APIs JSON

Laravel también provee algunos helpers para testear API JSON y sus respuestas. Por ejemplo, los métodos `json`, `get`, `post`, `put`, `patch`, y `delete` se pueden usar para emitir solicitudes con varios verbos HTTP. También se pueden pasar fácilmente datos y encabezados a estos métodos. Para comenzar, se tiene una prueba para hacer una solicitud `POST` a `/user` y validar que se devuelvan los datos esperados:

    <?php
    
    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->json('POST', '/user', ['name' => 'Sally']);
    
            $response
                ->assertStatus(200)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }
    

> {tip} El método `assertJson` convierte la respuesta a un array y utiliza `PHPUnit::assertArraySubset` para verificar que el array dado existe dentro de la respuesta JSON que fue devuelta por la aplicación. Entonces, si hay otras propiedades en la respuesta JSON, esta prueba aún pasará mientras el fragmento dado esté presente.

<a name="verifying-exact-match"></a>

### Verificación de una coincidencia JSON exacta

Si desea verificar que el array dado coincide de manera **exacta** para el JSON devuelto, debe usar el método `assertExactJson`:

    <?php
    
    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->json('POST', '/user', ['name' => 'Sally']);
    
            $response
                ->assertStatus(200)
                ->assertExactJson([
                    'created' => true,
                ]);
        }
    }
    

<a name="testing-file-uploads"></a>

## Pruebas de Subida de Archivos

La clase `Illuminate\Http\UploadedFile` provee un método `fake` que puede usarse para generar archivos vacíos o imágenes para probar. Esto, combinado con el método `fake` de la facade `Storage` simplifica enormente las pruebas de subidas de archivos. Por ejemplo, puedes combinar esas dos características para probar fácilmente un formulario de subida de un avatar:

    <?php
    
    namespace Tests\Feature;
    
    use Tests\TestCase;
    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    
    class ExampleTest extends TestCase
    {
        public function testAvatarUpload()
        {
            Storage::fake('avatars');
    
            $response = $this->json('POST', '/avatar', [
                'avatar' => UploadedFile::fake()->image('avatar.jpg')
            ]);
    
            // Assert the file was stored...
            Storage::disk('avatars')->assertExists('avatar.jpg');
    
            // Assert a file does not exist...
            Storage::disk('avatars')->assertMissing('missing.jpg');
        }
    }
    

#### Personalización de Archivo Falso

Cuando creas archivos usando el método `fake`, puedes especificar el ancho, la altura y el tamaño de la imagen para probar lo mejor posible tus reglas de validación:

    UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);
    

Además de crear imágenes, puedes crear archivos de cualquier otro formato usando el método `create`:

    UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);
    

<a name="available-assertions"></a>

## Verificaciones Disponibles

Laravel provee gran variedad de métodos de verificación para tus pruebas [PHPUnit](https://phpunit.de/). Se puede acceder a dichas verificaciones en la respuesta devuelta desde los métodos de prueba `json`, `get`, `post`, `put` y `delete`:

| Método                                                                                      | Descripción                                                                            |
| ------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| `$response->assertSuccessful();`                                                         | Verifica que la respuesta tiene un código de estado exitoso.                           |
| `$response->assertStatus($code);`                                                        | Verifica que la respuesta tiene el código dado.                                        |
| `$response->assertRedirect($uri);`                                                       | Verifica que la respuesta es una redirección a un URI determinado.                     |
| `$response->assertHeader($headerName, $value = null);`                                   | Verifica que la cabecera (header) esta presente en la respuesta.                       |
| `$response->assertCookie($cookieName, $value = null);`                                   | Verifica de que la respuesta contiene la cookie dada.                                  |
| `$response->assertPlainCookie($cookieName, $value = null);`                              | Verifica que la respuesta contiene la cookie dada (sin cifrar).                        |
| `$response->assertCookieExpired($cookieName);`                                           | Verifica de que la respuesta contiene la cookie dada y la misma está expirada.         |
| `$response->assertCookieMissing($cookieName);`                                           | Verifica de que la respuesta no contiene la cookie dada.                               |
| `$response->assertSessionHas($key, $value = null);`                                      | Verifica que la sesión contiene la información suministrada.                           |
| `$response->assertSessionHasErrors(array $keys, $format = null, $errorBag = 'default');` | Verifica que la sesión contiene un error para el campo dado.                           |
| `$response->assertSessionMissing($key);`                                                 | Verifica que la sesión no contiene la información dada.                                |
| `$response->assertJson(array $data);`                                                    | Verifica que la respuesta contiene los datos en JSON suministrados.                    |
| `$response->assertJsonFragment(array $data);`                                            | Verifica que la respuesta contiene un fragmento de los datos en JSON suministrados.    |
| `$response->assertJsonMissing(array $data);`                                             | Verifica que la respuesta no contiene un fragmento de los datos en JSON suministrados. |
| `$response->assertExactJson(array $data);`                                               | Verifica que la respuesta contiene una coincidencia exacta de los datos JSON dados.    |
| `$response->assertJsonStructure(array $structure);`                                      | Verifica que la respuesta tiene la estructura JSON dada.                               |
| `$response->assertViewIs($value);`                                                       | Verifica que la vista dada fue devuelta por la ruta.                                   |
| `$response->assertViewHas($key, $value = null);`                                         | Verifica que a la respuesta con una vista se le envió un dato.                         |
| `$response->assertViewHasAll(array $data);`                                              | Verifica que la vista de la respuesta tiene una lista de datos dada.                   |
| `$response->assertViewMissing($key);`                                                    | Verifica que a la vista que retorna la respuesta le falta una parte de los datos.      |
| `$response->assertSee($value);`                                                          | Verifica que la cadena dada está contenida dentro del texto de la respuesta.           |
| `$response->assertDontSee($value);`                                                      | Verifica que la cadena dada no se encuentra dentro del texto de la respuesta.          |
| `$response->assertSeeText($value);`                                                      | Verifica que la cadena dada está contenida dentro del texto de la respuesta.           |
| `$response->assertDontSeeText($value);`                                                  | Verifica que la cadena dada no está dentro del texto de la respuesta.                  |