# Tests En El Navegador (Laravel Dusk)

- [Introducción](#introduction)
- [Instalación](#installation) 
    - [Usar Otros Navegadores](#using-other-browsers)
- [Comenzando](#getting-started) 
    - [Generar Tests](#generating-tests)
    - [Ejecutar Pruebas](#running-tests)
    - [Manejo Del Ambiente](#environment-handling)
    - [Crear Navegadores](#creating-browsers)
    - [Autenticación](#authentication)
- [Interacción Con elementos](#interacting-with-elements) 
    - [Selectores Dusk](#dusk-selectors)
    - [Hacer Clic en Enlaces](#clicking-links)
    - [Texto, Valores & Atributos](#text-values-and-attributes)
    - [Usar Formularios](#using-forms)
    - [Adjuntar Archivos](#attaching-files)
    - [Usar El Teclado](#using-the-keyboard)
    - [Usar el Mouse](#using-the-mouse)
    - [Mantener el Ámbito De Selectores](#scoping-selectors)
    - [Esperar Por Elementos](#waiting-for-elements)
    - [Making Vue Assertions](#making-vue-assertions)
- [Available Assertions](#available-assertions)
- [Páginas](#pages) 
    - [Generar Páginas](#generating-pages)
    - [Configurar Páginas](#configuring-pages)
    - [Navegar Hacia Páginas](#navigating-to-pages)
    - [Shorthand Selectors](#shorthand-selectors)
    - [Métodos de Página](#page-methods)
- [Componentes](#components) 
    - [Generar Componentes](#generating-components)
    - [Usar Componentes](#using-components)
- [Integración Contínua](#continuous-integration) 
    - [Travis CI](#running-tests-on-travis-ci)
    - [CircleCI](#running-tests-on-circle-ci)
    - [Codeship](#running-tests-on-codeship)

<a name="introduction"></a>

## Introducción

Laravel Dusk provee una API de automatización y pruebas con el navegador que es fácil de usar. Por defecto, Dusk no requiere que se instale JDK o Selenium en la máquina. En su lugar, Dusk usa una instalación independiente de [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home). Sin embargo, se tiene la libertad de usar cualquier otro driver compatible con Selenium que se desee.

<a name="installation"></a>

## Instalación

Para comenzar, se debe incluir en el proyecto la dependencia de Composer `laravel/dusk`:

    composer require --dev laravel/dusk
    

Una vez que Dusk está instalado, se debe registrar el service provider `Laravel\Dusk\DuskServiceProvider`. Típicamente, esto se realizará automáticamente vía el registro automático de service provider de Laravel.

> {note} Si se registra manualmente el service provider de Dusk, **nunca** debería hacerlo en el servidor del ambiente de producción, ya que esto podría permitir a usuarios arbitrarios que se autentiquen contra la aplicación.

Después de instalar el paquete Dusk, ejecute el comando de Artisan `dusk:install`:

    php artisan dusk:install
    

Se creará un directorio `Browser` dentro del directorio `tests` y contendrá una prueba de ejemplo. Seguidamente, configure la variable de ambiente `APP_URL` en el archivo `.env`. Este valor debe coincidir con la URL que se usa para acceder a la aplicación desde un navegador.

Para ejecutar las pruebas, se usa el comando de Artisan `dusk`. El comando `dusk` acepta cualquier argumento que acepte el comando `phpunit`:

    php artisan dusk
    

<a name="using-other-browsers"></a>

### Usar Otros Navegadores

Por defecto, Dusk usa Google Chrome y una instalación independiente de [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home) para ejecutar las pruebas en el navegador. Sin embargo, se puede iniciar un servidor propio de Selenium y ejecutar los test en cualquier navegador que se desee.

Para comenzar, se abre el archivo `tests/DuskTestCase.php`, el cual es el caso base de tests para la aplicación. Dentro de este archivo, se puede eliminar la llamada al método `startChromeDriver`. Esto hará que Dusk no ejecute automáticamente el ChromeDriver:

    /**
     * Prepare for Dusk test execution.
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }
    

Seguidamente, se puede modificar el método `driver` para conectarse a la URL y el puerto que se quiera. Adicionalmente, se pueden modificar las "capacidades deseadas" que se deberían pasar al WebDriver:

    /**
     * Create the RemoteWebDriver instance.
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:4444/wd/hub', DesiredCapabilities::phantomjs()
        );
    }
    

<a name="getting-started"></a>

## Comenzando

<a name="generating-tests"></a>

### Generar Pruebas

Para generar una prueba de Dusk, se usa el comando de Artisan `dusk:make`. La prueba que se genera se coloca en el directorio `test/Browser`:

    php artisan dusk:make LoginTest
    

<a name="running-tests"></a>

### Ejecutar Pruebas

Para ejecutar las pruebas de browser, se usa el comando de Artisan `dusk`:

    php artisan dusk
    

El comando `dusk` acepta cualquier argumento que normalmente aceptaría el runner de PHPUnit, permitiendo ejecutar unicamente los test de un cierto [group](https://phpunit.de/manual/current/en/appendixes.annotations.html#appendixes.annotations.group), etc:

    php artisan dusk --group=foo
    

#### Comenzar Manualmente ChromeDriver

Por defecto, Dusk automáticamente intentará iniciar el ChromeDriver. Si no funciona para un sistema particular, se puede iniciar manualmente el ChromeDriver antes de ejecutar el comando `dusk`. Si se elige iniciar el ChromeDriver manualmente, se debe comentar la siguiente línea del archivo `tests/DuskTestCase.php`:

    /**
     * Prepare for Dusk test execution.
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }
    

Adicionalmente, si el ChromeDriver se inicia en un puerto distinto al 9515, se deber´+ia modificar el método `driver` de la misma clase:

    /**
     * Create the RemoteWebDriver instance.
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:9515', DesiredCapabilities::chrome()
        );
    }
    

<a name="environment-handling"></a>

### Manejo Del Ambiente

Para forzar a Dusk a usar su propio archivo de ambiente cuando se ejecutan las pruebas, se crea un archivo `.env.dusk.{environment}` en la raíz del proyecto. Por ejemplo, si se va a iniciar el comando `dusk` desde un ambiente `local`, se debería crear un archivo `.env.dusk.local`.

Cuando se ejecutan las pruebas, Dusk hará un respaldo del archivo `.env` y renombrará el ambiente de Dusk a `.env`. Una vez que se hayan completado, el archivo `.env` se restaurará.

<a name="creating-browsers"></a>

### Crear Navegadores

Para comenzar, escribamos una prueba que verifique que se puede iniciar sesión en la aplicación. Después de generar una prueba, se puede modificar para navegar a la página de incio de sesión, ingresar algunas credenciales y hacer clic en el botoón "Login". Para crear una instancia del navegador, se llama al método `browse`:

    <?php
    
    namespace Tests\Browser;
    
    use App\User;
    use Tests\DuskTestCase;
    use Laravel\Dusk\Chrome;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    
    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;
    
        /**
         * A basic browser test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $user = factory(User::class)->create([
                'email' => 'taylor@laravel.com',
            ]);
    
            $this->browse(function ($browser) use ($user) {
                $browser->visit('/login')
                        ->type('email', $user->email)
                        ->type('password', 'secret')
                        ->press('Login')
                        ->assertPathIs('/home');
            });
        }
    }
    

Como se puede ver en el ejemplo anterior, el método `browse` acepta un callback. Dusk pasa automáticamente una instancia del navegador a este callback y es el objeto principal que se usa para interactuar con la aplicación.

> {tip} Este test se puede usar para probar la pantalla de inicio de sesión que genera el comando de Artisan `make:auth`.

#### Crear Multiples Navegadores

En algunas ocasiones puede que se necesiten múltiples navegadores para realizar apropiadamente una prueba. Por ejemplo, se pueden necesitar múltiples navegadores para probar una pantalla de chat que interactue websockets. Para crear múltiples navegadores, simplemente se "pide" por más de uno en la firma del callback que se le da al método `browser`:

    $this->browse(function ($first, $second) {
        $first->loginAs(User::find(1))
              ->visit('/home')
              ->waitForText('Message');
    
        $second->loginAs(User::find(2))
               ->visit('/home')
               ->waitForText('Message')
               ->type('message', 'Hey Taylor')
               ->press('Send');
    
        $first->waitForText('Hey Taylor')
              ->assertSee('Jeffrey Way');
    });
    

#### Redimensionar Ventanas de Navegador

Se puede usar el método `resize` para ajustar el tamaño de la ventana del navegador:

    $browser->resize(1920, 1080);
    

El método `maximize` se puede usar para maximizar la ventana del navegador:

    $browser->maximize();
    

<a name="authentication"></a>

### Autenticación

Frecuentemente, se van a probar páginas que requieren autenticación. Se puede usar el método de Dusk `loginAs` para evitar interactuar con la pantalla de inicio de sesión durante cada test. El método `loginAs` acepta un ID de usuario o una instancia del modelo user:

    $this->browse(function ($first, $second) {
        $first->loginAs(User::find(1))
              ->visit('/home');
    });
    

> {note} Después de usar el método `loginAs`, la sesión del usuario se mantendrá para todos los test dentro del archivo.

<a name="interacting-with-elements"></a>

## Interacción Con elementos

<a name="dusk-selectors"></a>

### Selectores Dusk

Elegir buenos selectores CSS para interactuar con los elementos es una de las partes más difíciles cuando se escriben las pruebas con Dusk. Con el tiempo, los ambios en el frontend pueden causar que selectores CSS como los siguientes rompan los tests:

    // HTML...
    
    <button>Login</button>
    
    // Test...
    
    $browser->click('.login-page .container div > button');
    

Los selectores de Dusk permiten enfocarse en escribir test efectivos en lugar de recordar selectores de CSS. Para definir un selector, se agrega un atributo `dusk` al elemento HTML. Luego, se pone el prefijo `@` en el selector para manipular el elemento adjunto dentro las pruebas de Dusk:

    // HTML...
    
    <button dusk="login-button">Login</button>
    
    // Test...
    
    $browser->click('@login-button');
    

<a name="clicking-links"></a>

### Hacer Clic en Enlaces

Para hacer clic en un enlace, se puede usar el método `clickLink` en la instancia del navegador. El método `clickLink` hará clic en el enlace que muestre el texto dado:

    $browser->clickLink($linkText);
    

> {note} Este método interactúa con jQuery. Si jQuery no está disponible, Dusk lo inyectará automáticamente a la página para que se encuentre disponible durante la duración de la prueba.

<a name="text-values-and-attributes"></a>

### Texto, Valores & Atributos

#### Recuperar & Establecer Valores

Dusk provee varios métodos para interactuar con el texto que se está mostrando, valores y atributos de los elementos de la página. Por ejemplo, para obtener el "value" de un elemento que coincida con un selector específico, se usa el método `value`:

    // Retrieve the value...
    $value = $browser->value('selector');
    
    // Set the value...
    $browser->value('selector', 'value');
    

#### Recuperar Texto

El método `text` se puede usar para recupera el texto que muestra un elemento que coincida con el selector dado:

    $text = $browser->text('selector');
    

#### Recuperar Atributos

Finalmente, el método `attribute` se puede usar para recuperar un atributo del elemento que coincida con el selector dado:

    $attribute = $browser->attribute('selector', 'value');
    

<a name="using-forms"></a>

### Usar Formularios

#### Transcribir Valores

Dusk provee una variedad de métodos para interactuar con formularios y elementos de captura de datos. Primero, veamos un ejemplo de transcribir texto en un campo:

    $browser->type('email', 'taylor@laravel.com');
    

Nótese que, aunque el metodo acepta uno si es necesario, no se requiere pasar un selector CSS al método `type`. Si no se provee un selector CSS, Dusk buscará un elemento con el atributo `name` dado. Finalmente, Dusk intentará encontrar un `textarea` con el atributo `name` dado.

Para incluir texto en un campo sin borrar su contenido, se puede usar el método `append`:

    $browser->type('tags', 'foo')
            ->append('tags', ', bar, baz');
    

Se puede limpiar el valor de un input usando el método `clear`:

    $browser->clear('email');
    

#### Menús Desplegables

Para seleccionar un valor en un menú desplegable, se puede usar el método `select`. Como el método `type`, el método `select` no requiere el uso de un selector CSS. Cuando se pasa un valor al método `select`, se debe enviar el valor subyacente de la opción y no el texto que se muestra:

    $browser->select('size', 'Large');
    

Se puede seleccionar un valor aleatorio omitiendo el segundo parámetro:

    $browser->select('size');
    

#### Casillas De Selección

Para "chequear" una casilla de selección, se puede usar el método `check`. Como en muchos métodos relacionados con el ingreso de datos, no se requiere un selector CSS completo. Si no se puede encontrar una coincidencia exacta con el selector, Dusk buscará una casilla de verificación que coincida con el atributo `name`:

    $browser->check('terms');
    
    $browser->uncheck('terms');
    

#### Botones de Radio

Para "seleccionar" una opción de un botón de radio, se puede usar el método `radio`. Como muchos otros métodos de ingreso de datos, no se requiere un selector CSS completo. Si no se puede encontrar una coincidencia exacta con el selector, Dusk buscará un botón de radio que coincida con los atributos `name` y `value`:

    $browser->radio('version', 'php7');
    

<a name="attaching-files"></a>

### Adjuntar Archivos

El método `attach` se puede usar para adjuntar un archivo a un elemento `file`. Como en muchos métodos relacionados con el ingreso de datos, no se requiere un selector CSS completo. Si no se puede encontrar una coincidencia exacta con el selector, Dusk buscará un campo de archivo que coincida con el atributo `name`:

    $browser->attach('photo', __DIR__.'/photos/me.png');
    

<a name="using-the-keyboard"></a>

### Usar El Teclado

El método `keys` permite proveer secuencias de entrada más complejas de las que normalmente se permiten con el método `type`. Por ejemplo, se pueden presionar teclas modificadoras mientras se ingresan valores. En este ejemplo, la tecla `shift` se mantiene presionada mientras se ingresa `taylor` en el elemento que coincide con el selector dado. Después de escribir `taylor`, se escribe `otwell` sin ninguna tecla modificadora:

    $browser->keys('selector', ['{shift}', 'taylor'], 'otwell');
    

Se puede incluso enviar una "hot key" al selector primario que contiene la aplicación:

    $browser->keys('.app', ['{command}', 'j']);
    

> {tip} Todas las teclas modificadoras estan rodeadas por caracteres `{}`, y coinciden con las constantes definidas en la clase `Facebook\WebDriver\WebDriverKeys`, la cual se puede [encontrar en Github](https://github.com/facebook/php-webdriver/blob/community/lib/WebDriverKeys.php).

<a name="using-the-mouse"></a>

### Usar el Mouse

#### Hacer Clic En Elementos

El método `click` se puede usar para hacer "clic" en el elemento que coincida con el selector dado:

    $browser->click('.selector');
    

#### Mouseover

El método `mouseover` se puede usar para cuando se necesita mover el mouse sobre el elemento que coincida con el selector:

    $browser->mouseover('.selector');
    

#### Arrastrar & Soltar

El método `drag` se puede usar para arrastrar un elelmento que coincida con el selector dado hacia otro elemento:

    $browser->drag('.from-selector', '.to-selector');
    

O, se puede arrastrar un elemento en una ciertas dirección:

    $browser->dragLeft('.selector', 10);
    $browser->dragRight('.selector', 10);
    $browser->dragUp('.selector', 10);
    $browser->dragDown('.selector', 10);
    

<a name="scoping-selectors"></a>

### Mantener el Ámbito De Selectores

En ocasiones, se puede querer ejecutar varias operaciones mientras se mantiene el alcance de dichas operaciones dentro de un selector dado. Por ejemplo, Se puede querer verificar que un determinado texto exista sólo dentro de una tabla y hacer clic en un botón dentro de esa misma tabla. Se puede usar el método `with` para lograr eso. Todas las operaciones que se ejecuten dentro de la función anónima que se le da al método `with` se mantendrán dentro del ámbito del selector original:

    $browser->with('.table', function ($table) {
        $table->assertSee('Hello World')
              ->clickLink('Delete');
    });
    

<a name="waiting-for-elements"></a>

### Esperar Por Elementos

Cuando se prueban aplicaciones que usan JavaScript de manera extensiva, frecuentemente se hace necesario "esperar" que ciertos elementos o datos estén disponibles antes de proseguir con la prueba. Dusk hace que esto sea muy sencillo. Usando una variedad de métodos, se puede esperar hasta que un elemento se haga visible en la página o inclusive esperar hasta que una expresión de JavaScript se evalúe como `true`.

#### Esperar

Si es necesario pausar la prueba por un determinado número de milisegundos, se usa el método `pause`:

    $browser->pause(1000);
    

#### Esperar Por Elementos

El método `wait` se puede usar para pausar la ejecución de una prueba hasta que el elemento que coincida con el selector CSS dado se muestre en la pantalla. Por defecto, esto hará que el test se detenga hasta por un máximo de cinco segundos antes de lanzar una excepción. Si es necesario, se puede pasar un período de espera personalizado como segundo parámetro al método:

    // Wait a maximum of five seconds for the selector...
    $browser->waitFor('.selector');
    
    // Wait a maximum of one second for the selector...
    $browser->waitFor('.selector', 1);
    

También se puede esperar hasta que el elemento seleccionado desaparezca de la página:

    $browser->waitUntilMissing('.selector');
    
    $browser->waitUntilMissing('.selector', 1);
    

#### Scoping Selectors When Available

En ocasiones, se quiere esperar por un selector dado e interactuar con el elemento que coincide con el selector. Por ejemplo, se puede querer esperar a que una ventana modal esté disponible y luego presionar el botón "OK" dentro de la modal. El método `whenAvailable` se puede usar en ese caso. Todas las operaciones de elementos que se realicen dentro de la función anónima dada tendrán el ámbito del selector original:

    $browser->whenAvailable('.modal', function ($modal) {
        $modal->assertSee('Hello World')
              ->press('OK');
    });
    

#### Esperar Por Texto

El método `waitForText` se puede usar para esperar hasta que el texto dado se muestre en la página:

    // Wait a maximum of five seconds for the text...
    $browser->waitForText('Hello World');
    
    // Wait a maximum of one second for the text...
    $browser->waitForText('Hello World', 1);
    

#### Esperar Por Enlaces

El método `waitForLink` se puede usar para esperar hasta que el texto de enlace dado se muestre en la página:

    // Wait a maximum of five seconds for the link...
    $browser->waitForLink('Create');
    
    // Wait a maximum of one second for the link...
    $browser->waitForLink('Create', 1);
    

#### Esperar Por La Dirección De La Página

Cuando se hace una revisión de una ruta tal como `$browser->assertPathIs('/home')`, esta puede fallar si `window.location.pathname` se está actualizando asíncronamente. Se puede usar el método `waitForLocation` para esperar a que la dirección tenga un valor dado:

    $browser->waitForLocation('/secret');
    

#### Esperar Por La Recarga De La Página

Si es necesario hacer verificaciones después de que la página se hata recargado, se usa el método `waitForReload`:

    $browser->click('.some-action')
            ->waitForReload()
            ->assertSee('something');
    

#### Esperar Por Expresiones JavaScript

Algunas veces se quiere pausar le ejecución de una prueba hasta que una determinada expresión JavaScript se evalue como `true`. Esto se puede lograr fácilmente usando el método `waitUntil`. Cuando se pasa una expresión a este método, no es necesario incluir la palabra reservada `return` ni terminar con un punto y coma:

    // Wait a maximum of five seconds for the expression to be true...
    $browser->waitUntil('App.dataLoaded');
    
    $browser->waitUntil('App.data.servers.length > 0');
    
    // Wait a maximum of one second for the expression to be true...
    $browser->waitUntil('App.data.servers.length > 0', 1);
    

#### Esperar Con Una Función Anónima

Muchos de los métodos "wait" en Dusk se basan en el método subyacente `waitUsing`. Se puede usar este método directamente para esperar a que una determinada funciión anónima retorne `true`. El método `waitUsing` acepta el máximo número de segundos a esperar, el intervalo en el cual la función va a ser evaluada, la función anónima, y un mensaje opcional en caso de fallo:

    $browser->waitUsing(10, 1, function () use ($something) {
        return $something->isReady();
    }, "Something wasn't ready in time.");
    

<a name="making-vue-assertions"></a>

### Hacer Verificaciones Vue

Dusk permite hacer verificaciones sobre el estado de los componentes de datosd de [Vue](https://vuejs.org). Por ejemplo, imaginando que la aplicación contiene el siguiente componete de Vue:

    // HTML...
    
    <profile dusk="profile-component"></profile>
    
    // Component Definition...
    
    Vue.component('profile', {
        template: '<div>{{ user.name }}</div>',
    
        data: function () {
            return {
                user: {
                  name: 'Taylor'
                }
            };
        }
    });
    

Se puede verificar el estado del componente de Vue de la siguiente forma:

    /**
     * A basic Vue test example.
     *
     * @return void
     */
    public function testVue()
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/')
                    ->assertVue('user.name', 'Taylor', '@profile-component');
        });
    }
    

<a name="available-assertions"></a>

## Verificaciones Disponibles

Dusk provee una variedad de verificaciones que se pueden hacer contra la aplicación. Todas las verificaciones disponibles están documentadas en la siguiente tabla:

| Assertion                                                    | Descripción                                                                                     |
| ------------------------------------------------------------ | ----------------------------------------------------------------------------------------------- |
| `$browser->assertTitle($title)`                           | Verificar que el título de la página coincida con el texto dado.                                |
| `$browser->assertTitleContains($title)`                   | Verificar que el título de la página contenga el texto dado.                                    |
| `$browser->assertPathBeginsWith($path)`                   | Verificar que la ruta URL actual comience con la ruta dada.                                     |
| `$browser->assertPathIs('/home')`                         | Verificar que la ruta actual coincida con la ruta dada.                                         |
| `$browser->assertPathIsNot('/home')`                      | Verificar que la ruta actual no coincida con la ruta dada.                                      |
| `$browser->assertRouteIs($name, $parameters)`             | Verificar que la URL actual coincida con la URL de la ruta con nombre dada.                     |
| `$browser->assertQueryStringHas($name, $value)`           | Assert the given query string parameter is present and has a given value.                       |
| `$browser->assertQueryStringMissing($name)`               | Assert the given query string parameter is missing.                                             |
| `$browser->assertHasQueryStringParameter($name)`          | Assert that the given query string parameter is present.                                        |
| `$browser->assertHasCookie($name)`                        | Verificar que la cookie dada está presente.                                                     |
| `$browser->assertCookieMissing($name)`                    | Verificar que la cookie dada no está presente.                                                  |
| `$browser->assertCookieValue($name, $value)`              | Verificar que una cookie tiene un valor determinado.                                            |
| `$browser->assertPlainCookieValue($name, $value)`         | Verificar que una cookie no encriptada tiene un valor determinado.                              |
| `$browser->assertSee($text)`                              | Verificar que el texto dado está presente en la página.                                         |
| `$browser->assertDontSee($text)`                          | Verificar que el texto dado no está presente en la página.                                      |
| `$browser->assertSeeIn($selector, $text)`                 | Verificar que el texto dado está presente dentro de un selector.                                |
| `$browser->assertDontSeeIn($selector, $text)`             | Verificar que el texto dado no esta presente dentro del selector.                               |
| `$browser->assertSourceHas($code)`                        | Verificar que determinado código fuente está presente en la página.                             |
| `$browser->assertSourceMissing($code)`                    | Verificar que determinado código fuente no está presente en la página.                          |
| `$browser->assertSeeLink($linkText)`                      | Verificar que el enlace dado está presente en la página.                                        |
| `$browser->assertDontSeeLink($linkText)`                  | Verificar que el enlace dado no está presente en la página.                                     |
| `$browser->assertInputValue($field, $value)`              | Verificar que la casilla de texto dada tiene un valor determinado.                              |
| `$browser->assertInputValueIsNot($field, $value)`         | Verificar que la casilla de texto no tiene el valor dado.                                       |
| `$browser->assertChecked($field)`                         | Verificar que la casilla de verificación dada está marcada.                                     |
| `$browser->assertNotChecked($field)`                      | Verificar que la casilla de verificación dada no está marcada.                                  |
| `$browser->assertRadioSelected($field, $value)`           | Verificar que el botón de radio dado está seleccionado.                                         |
| `$browser->assertRadioNotSelected($field, $value)`        | Verificar que el botón de radio dado no está seleccionado.                                      |
| `$browser->assertSelected($field, $value)`                | Verificar que determinado menú desplegable dado tiene seleccionado el valor dado.               |
| `$browser->assertNotSelected($field, $value)`             | Verificar que determinado menú desplegable no tiene seleccionado el valor dado.                 |
| `$browser->assertSelectHasOptions($field, $values)`       | Verificar que los valores en la matriz dada están disponibles para ser seleccionados.           |
| `$browser->assertSelectMissingOptions($field, $values)`   | Verificar que los valores en la matriz dada no están disponibles para ser seleccionados.        |
| `$browser->assertSelectHasOption($field, $value)`         | Verificar que el valor dado está disponible para ser seleccionado en el campo dado.             |
| `$browser->assertValue($selector, $value)`                | Verificar que el elemento que coincide con el selector dado tiene el valor dado.                |
| `$browser->assertVisible($selector)`                      | Verificar que el elemento que coincide con el selector es visible.                              |
| `$browser->assertMissing($selector)`                      | Verificar que el elemento que coincide con el selector no es visible.                           |
| `$browser->assertDialogOpened($message)`                  | Verificar que se ha abierto una ventana de diálogo JavaScript con el mensaje dado.              |
| `$browser->assertVue($property, $value, $component)`      | Verificar que la propiedad data de un determinado componente Vue coincide con el valor dado.    |
| `$browser->assertVueIsNot($property, $value, $component)` | Verificar que la propiedad data de un componente Vue determinado no coincide con el valor dado. |

<a name="pages"></a>

## Páginas

En ocasiones, la prueba requiere que se hagan en secuencia varias acciones complicadas. Esto puede hacer que las pruebas sean difíciles de leer y entender. Las Páginas permiten definir acciones expresivas que luego pueden ser realizadas en una página determinada usando un solo método. Las Páginas también permiten definir atajos hacia selectores comunes en la aplicación o en una sola página.

<a name="generating-pages"></a>

### Generar Páginas

Para generar un objeto de página, se usa el comando de Artisan `dusk:page`. Todos estos objetos se se colocarán en el directorio `tests/Browser/Pages`:

    php artisan dusk:page Login
    

<a name="configuring-pages"></a>

### Configurar Páginas

Por defecto, las páginas tienen tres métodos: `url`, `assert`, y `elements`. Discutiremos ahora los métodos `url` y `assert`. El método `elements` se discute con [más detalle abajo](#shorthand-selectors).

#### El Método `url`

El método `url` debería retornar la ruta a la URL que representa la página. Dusk usa esta URL cuando navega la página en el browser:

    /**
     * Get the URL for the page.
     *
     * @return string
     */
    public function url()
    {
        return '/login';
    }
    

#### El Método `assert`

El método `assert` puede tomar cualquier verificación necesaria para verificar que el navegador está realmente en la página dada. Completar este método no es necesario, sin embargo, se tiene la libertad de hacer estas verificaciones si se lo desea. Estas verificaciones se ejecutan automáticamente cuando se navega en la página:

    /**
     * Assert that the browser is on the page.
     *
     * @return void
     */
    public function assert(Browser $browser)
    {
        $browser->assertPathIs($this->url());
    }
    

<a name="navigating-to-pages"></a>

### Navegar Hacia Páginas

Una vez que la página ha sido configurada, se puede navegar usando el método `visit`:

    use Tests\Browser\Pages\Login;
    
    $browser->visit(new Login);
    

En ocasiones uno ya se encuentra en una página dada y se necesita "cargar" los selectores y métodos en el contexto actual de la prueba. Esto es común cuando se presiona un botón y se es redirigido a una página determinada sin navegar explícitamente hacia ella. En esta situación, se puede usar el método `on` para cargar la página:

    use Tests\Browser\Pages\CreatePlaylist;
    
    $browser->visit('/dashboard')
            ->clickLink('Create Playlist')
            ->on(new CreatePlaylist)
            ->assertSee('@create');
    

<a name="shorthand-selectors"></a>

### Shorthand Selectors

The `elements` method of pages allows you to define quick, easy-to-remember shortcuts for any CSS selector on your page. For example, let's define a shortcut for the "email" input field of the application's login page:

    /**
     * Get the element shortcuts for the page.
     *
     * @return array
     */
    public function elements()
    {
        return [
            '@email' => 'input[name=email]',
        ];
    }
    

Now, you may use this shorthand selector anywhere you would use a full CSS selector:

    $browser->type('@email', 'taylor@laravel.com');
    

#### Global Shorthand Selectors

After installing Dusk, a base `Page` class will be placed in your `tests/Browser/Pages` directory. This class contains a `siteElements` method which may be used to define global shorthand selectors that should be available on every page throughout your application:

    /**
     * Get the global element shortcuts for the site.
     *
     * @return array
     */
    public static function siteElements()
    {
        return [
            '@element' => '#selector',
        ];
    }
    

<a name="page-methods"></a>

### Métodos de Página

In addition to the default methods defined on pages, you may define additional methods which may be used throughout your tests. For example, let's imagine we are building a music management application. A common action for one page of the application might be to create a playlist. Instead of re-writing the logic to create a playlist in each test, you may define a `createPlaylist` method on a page class:

    <?php
    
    namespace Tests\Browser\Pages;
    
    use Laravel\Dusk\Browser;
    
    class Dashboard extends Page
    {
        // Other page methods...
    
        /**
         * Create a new playlist.
         *
         * @param  \Laravel\Dusk\Browser  $browser
         * @param  string  $name
         * @return void
         */
        public function createPlaylist(Browser $browser, $name)
        {
            $browser->type('name', $name)
                    ->check('share')
                    ->press('Create Playlist');
        }
    }
    

Once the method has been defined, you may use it within any test that utilizes the page. The browser instance will automatically be passed to the page method:

    use Tests\Browser\Pages\Dashboard;
    
    $browser->visit(new Dashboard)
            ->createPlaylist('My Playlist')
            ->assertSee('My Playlist');
    

<a name="components"></a>

## Componentes

Components are similar to Dusk’s “page objects”, but are intended for pieces of UI and functionality that are re-used throughout your application, such as a navigation bar or notification window. As such, components are not bound to specific URLs.

<a name="generating-components"></a>

### Generar Componentes

To generate a component, use the `dusk:component` Artisan command. New components are placed in the `test/Browser/Components` directory:

    php artisan dusk:component DatePicker
    

As shown above, a "date picker" is an example of a component that might exist throughout your application on a variety of pages. It can become cumbersome to manually write the browser automation logic to select a date in dozens of tests throughout your test suite. Instead, we can define a Dusk component to represent the date picker, allowing us to encapsulate that logic within the component:

    <?php
    
    namespace Tests\Browser\Components;
    
    use Laravel\Dusk\Browser;
    use Laravel\Dusk\Component as BaseComponent;
    
    class DatePicker extends BaseComponent
    {
        /**
         * Get the root selector for the component.
         *
         * @return string
         */
        public function selector()
        {
            return '.date-picker';
        }
    
        /**
         * Assert that the browser page contains the component.
         *
         * @param  Browser  $browser
         * @return void
         */
        public function assert(Browser $browser)
        {
            $browser->assertVisible($this->selector());
        }
    
        /**
         * Get the element shortcuts for the component.
         *
         * @return array
         */
        public function elements()
        {
            return [
                '@date-field' => 'input.datepicker-input',
                '@month-list' => 'div > div.datepicker-months',
                '@day-list' => 'div > div.datepicker-days',
            ];
        }
    
        /**
         * Select the given date.
         *
         * @param  \Laravel\Dusk\Browser  $browser
         * @param  int  $month
         * @param  int  $year
         * @return void
         */
        public function selectDate($browser, $month, $year)
        {
            $browser->click('@date-field')
                    ->within('@month-list', function ($browser) use ($month) {
                        $browser->click($month);
                    })
                    ->within('@day-list', function ($browser) use ($day) {
                        $browser->click($day);
                    });
        }
    }
    

<a name="using-components"></a>

### Usar Componentes

Once the component has been defined, we can easily select a date within the date picker from any test. And, if the logic necessary to select a date changes, we only need to update the component:

    <?php
    
    namespace Tests\Browser;
    
    use Tests\DuskTestCase;
    use Laravel\Dusk\Browser;
    use Tests\Browser\Components\DatePicker;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    
    class ExampleTest extends DuskTestCase
    {
        /**
         * A basic component test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->browse(function (Browser $browser) {
                $browser->visit('/')
                        ->within(new DatePicker, function ($browser) {
                            $browser->selectDate(1, 2018);
                        })
                        ->assertSee('January');
            });
        }
    }
    

<a name="continuous-integration"></a>

## Integración Contínua

<a name="running-tests-on-travis-ci"></a>

### Travis CI

To run your Dusk tests on Travis CI, we will need to use the "sudo-enabled" Ubuntu 14.04 (Trusty) environment. Since Travis CI is not a graphical environment, we will need to take some extra steps in order to launch a Chrome browser. In addition, we will use `php artisan serve` to launch PHP's built-in web server:

    sudo: required
    dist: trusty
    
    addons:
       chrome: stable
    
    install:
    
       - cp .env.testing .env
       - travis_retry composer install --no-interaction --prefer-dist --no-suggest
    
    before_script:
    
       - google-chrome-stable --headless --disable-gpu --remote-debugging-port=9222 http://localhost &
       - php artisan serve &
    
    script:
    
       - php artisan dusk
    

<a name="running-tests-on-circle-ci"></a>

### CircleCI

#### CircleCI 1.0

If you are using CircleCI 1.0 to run your Dusk tests, you may use this configuration file as a starting point. Like TravisCI, we will use the `php artisan serve` command to launch PHP's built-in web server:

    dependencies:
      pre:
    
          - curl -L -o google-chrome.deb https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
          - sudo dpkg -i google-chrome.deb
          - sudo sed -i 's|HERE/chrome\"|HERE/chrome\" --disable-setuid-sandbox|g' /opt/google/chrome/google-chrome
          - rm google-chrome.deb
    
    test:
        pre:
    
            - "./vendor/laravel/dusk/bin/chromedriver-linux":
                background: true
            - cp .env.testing .env
            - "php artisan serve":
                background: true
    
        override:
    
            - php artisan dusk
    

#### CircleCI 2.0

If you are using CircleCI 2.0 to run your Dusk tests, you may add these steps to your build:

     version: 2
     jobs:
         build:
             steps:
    
                - run: sudo apt-get install -y libsqlite3-dev
                - run: cp .env.testing .env
                - run: composer install -n --ignore-platform-reqs
                - run: npm install
                - run: npm run production
                - run: vendor/bin/phpunit
    
                - run:
                   name: Start Chrome Driver
                   command: ./vendor/laravel/dusk/bin/chromedriver-linux
                   background: true
    
                - run:
                   name: Run Laravel Server
                   command: php artisan serve
                   background: true
    
                - run:
                   name: Run Laravel Dusk Tests
                   command: php artisan dusk
    

<a name="running-tests-on-codeship"></a>

### Codeship

To run Dusk tests on [Codeship](https://codeship.com), add the following commands to your Codeship project. Of course, these commands are simply a starting point and you are free to add additional commands as needed:

    phpenv local 7.1
    cp .env.testing .env
    composer install --no-interaction
    nohup bash -c "./vendor/laravel/dusk/bin/chromedriver-linux 2>&1 &"
    nohup bash -c "php artisan serve 2>&1 &" && sleep 5
    php artisan dusk