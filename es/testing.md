# Testing: Primeros pasos

- [Introducción](#introduction)
- [Entorno](#environment)
- [Creando & Ejecutando Tests](#creating-and-running-tests)

<a name="introduction"></a>

## Introducción

Laravel ha sido desarrollado teniendo en cuenta el testing. De hecho, el soporte para pruebas con PHPUnit se incluye "Out of the box" (de serie) y un archivo `phpunit.xml` ya está configurado para su aplicación. El framework también incluye métodos convenientes que le permiten probar sus aplicaciones de forma expresa.

De forma predeterminada, el directorio `tests` de su aplicación contiene dos directorios: `Feature` y `Unit`. "Unit test" son pruebas o "tests" que se enfocan en una porción muy pequeña y aislada de su código. De hecho, la mayoría de las pruebas unitarias probablemente se centran en un solo método. Las pruebas de funcionalidad o "feature tests" pueden probar una porción más grande de su código, incluyendo cómo varios objetos interactúan entre sí o incluso una solicitud HTTP completa a un "endpoint" JSON.

El archivo `ExampleTest.php` se incluye los directorios `Feature` y `Unit`. Después de instalar una nueva aplicación Laravel, simplemente ejecute `phpunit` en la línea de comandos para ejecutar sus pruebas.

<a name="environment"></a>

## Entorno

Cuando se ejecutan pruebas a través de `phpunit`, Laravel ajustará automáticamente el entorno de configuración a `testing` debido a las variables de entorno definidas en el fichero `phpunit. xml`. Laravel también configura automáticamente la sesión y la caché en el driver `array` mientras se realizan las pruebas, lo que significa que no se mantendrán los datos de la sesión o la caché durante la ejecución.

Puede definir libremente otros valores de configuración del entorno de prueba según sea necesario. Las variables de entorno `testing` pueden configurarse en el fichero `phpunit.xml`, ¡pero asegúrese de borrar su caché de configuración usando el comando Artisan `config:clear` antes de ejecutar sus pruebas!

<a name="creating-and-running-tests"></a>

## Creando & Ejecutando Tests

Para crear un nuevo test, utilizar el comando de Artisan `make:test`:

    // Create a test in the Feature directory...
    php artisan make:test UserTest
    
    // Create a test in the Unit directory...
    php artisan make:test UserTest --unit
    

Una vez generada la prueba, puede definir los métodos de prueba como lo haría normalmente con PHPUnit. Para ejecutar las pruebas, simplemente ejecute el comando `phpunit` desde su terminal:

    <?php
    
    namespace Tests\Unit;
    
    use Tests\TestCase;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    
    class ExampleTest extends TestCase
    {
        /**
         * A basic test example.
         *
         * @return void
         */
        public function testBasicTest()
        {
            $this->assertTrue(true);
        }
    }
    

> {note} Si define su propio método `setUp` dentro de una clase de prueba, asegúrese de llamar `parent::setUp()`.