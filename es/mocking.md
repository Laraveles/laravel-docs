# Mocking

- [Introducción](#introduction)
- [Fake Bus](#bus-fake)
- [Fake Event](#event-fake)
- [Fake Mail](#mail-fake)
- [Notificaciones](#notification-fake)
- [Fake Queue](#queue-fake)
- [Fake Storage](#storage-fake)
- [Facades](#mocking-facades)

<a name="introduction"></a>

## Introducción

Al probar las aplicaciones de Laravel, es posible que desee "simular" ciertos aspectos de su aplicación para que no se ejecuten realmente durante una prueba determinada. Por ejemplo, cuando se prueba un controlador que distribuye un evento, es posible que desee simular los detectores de eventos para que no se ejecuten realmente durante la prueba. Esto le permite probar únicamente la respuesta HTTP del controlador sin preocuparse por la ejecución de los detectores de eventos, ya que los detectores de eventos se pueden probar en su propio caso de prueba.

Laravel proporciona ayudantes para simular eventos, trabajos, y facades listos para usarse. Estos ayudantes proporcionan principalemente una capa conveniente sobre Mockery para que no tenga que realizar manualmente llamadas de método de Mockery complicadas. Por supuesto, es libre de usar [Mockery](http://docs.mockery.io/en/latest/) o la PHPUnit para crear sus propios simuladores o espías.

<a name="bus-fake"></a>

## Fake Bus

Como alternativa a la simulación, puede usar el método de facade `Bus` `fake` para evitar que se envíen trabajos. Cuando usa fakes, las afirmaciones se realizan después de que se ejecuta el código bajo prueba:

    <?php
    
    namespace Tests\Feature;
    
    use Tests\TestCase;
    use App\Jobs\ShipOrder;
    use Illuminate\Support\Facades\Bus;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    
    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Bus::fake();
    
            // Perform order shipping...
    
            Bus::assertDispatched(ShipOrder::class, function ($job) use ($order) {
                return $job->order->id === $order->id;
            });
    
            // Assert a job was not dispatched...
            Bus::assertNotDispatched(AnotherJob::class);
        }
    }
    

<a name="event-fake"></a>

## Fake Event

Como alternativa a la simulación, puede usar el método de facade `fake` `Bus`para evitar que se ejecuten los detectores de eventos. A continuación, puede afirmar que los eventos fueron enviados e incluso inspeccionar los datos que recibieron. Cuando usa fakes, las afirmaciones se realizan después de que se ejecuta el código bajo prueba:

    <?php
    
    namespace Tests\Feature;
    
    use Tests\TestCase;
    use App\Events\OrderShipped;
    use App\Events\OrderFailedToShip;
    use Illuminate\Support\Facades\Event;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    
    class ExampleTest extends TestCase
    {
        /**
         * Test order shipping.
         */
        public function testOrderShipping()
        {
            Event::fake();
    
            // Perform order shipping...
    
            Event::assertDispatched(OrderShipped::class, function ($e) use ($order) {
                return $e->order->id === $order->id;
            });
    
            Event::assertNotDispatched(OrderFailedToShip::class);
        }
    }
    

<a name="mail-fake"></a>

## Fake Mail

Puede usar el método de facade `fake` `Mail` para evitar que se envíe el correo. A continuación, puede afirmar que se enviaron [mailables](/docs/{{version}}/mail) a los usuarios e incluso inspeccionar los datos que ellos recibieron. Cuando usa fakes, las afirmaciones se realizan después de que se ejecuta el código bajo prueba:

    <?php
    
    namespace Tests\Feature;
    
    use Tests\TestCase;
    use App\Mail\OrderShipped;
    use Illuminate\Support\Facades\Mail;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    
    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Mail::fake();
    
            // Perform order shipping...
    
            Mail::assertSent(OrderShipped::class, function ($mail) use ($order) {
                return $mail->order->id === $order->id;
            });
    
            // Assert a message was sent to the given users...
            Mail::assertSent(OrderShipped::class, function ($mail) use ($user) {
                return $mail->hasTo($user->email) &&
                       $mail->hasCc('...') &&
                       $mail->hasBcc('...');
            });
    
            // Assert a mailable was sent twice...
            Mail::assertSent(OrderShipped::class, 2);
    
            // Assert a mailable was not sent...
            Mail::assertNotSent(AnotherMailable::class);
        }
    }
    

Si está haciendo queue en formato mailable para la entrega en segundo plano, debe usar el método `assertQueued` en vez de `assertSent`:

    Mail::assertQueued(...);
    Mail::assertNotQueued(...);
    

<a name="notification-fake"></a>

## Notificación Fake

Debe usar el método de facade `Notification` `fake` para evitar que se envíen las notificaciones. A continuación, puede afirmar que se enviaron las [notifications](/docs/{{version}}/notifications) a los usuarios e incluso inspeccionar los datos que ellos recibieron. Cuando usa fakes, las afirmaciones se realizan después de que se ejecuta el código bajo prueba:

    <?php
    
    namespace Tests\Feature;
    
    use Tests\TestCase;
    use App\Notifications\OrderShipped;
    use Illuminate\Support\Facades\Notification;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    
    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Notification::fake();
    
            // Perform order shipping...
    
            Notification::assertSentTo(
                $user,
                OrderShipped::class,
                function ($notification, $channels) use ($order) {
                    return $notification->order->id === $order->id;
                }
            );
    
            // Assert a notification was sent to the given users...
            Notification::assertSentTo(
                [$user], OrderShipped::class
            );
    
            // Assert a notification was not sent...
            Notification::assertNotSentTo(
                [$user], AnotherNotification::class
            );
        }
    }
    

<a name="queue-fake"></a>

## Fake Queue

Como alternativa a la simulación, puede usar el método de facade `fake` `Queue`para evitar que los trabajos se mantengan en cola. Puede entonces afirmar que los trabajos se enviaron a la cola e incluso los datos que recibieron. Cuando usa fakes, las afirmaciones se realizan después de que se ejecuta el código bajo prueba:

    <?php
    
    namespace Tests\Feature;
    
    use Tests\TestCase;
    use App\Jobs\ShipOrder;
    use Illuminate\Support\Facades\Queue;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    
    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Queue::fake();
    
            // Perform order shipping...
    
            Queue::assertPushed(ShipOrder::class, function ($job) use ($order) {
                return $job->order->id === $order->id;
            });
    
            // Assert a job was pushed to a given queue...
            Queue::assertPushedOn('queue-name', ShipOrder::class);
    
            // Assert a job was pushed twice...
            Queue::assertPushed(ShipOrder::class, 2);
    
            // Assert a job was not pushed...
            Queue::assertNotPushed(AnotherJob::class);
        }
    }
    

<a name="storage-fake"></a>

## Fake Storage

El método de facade `fake` `Storage` le permite generar fácilmente un disco fake que, combinado con las utilidades de generación de archivos de la clase `UploadedFile`, simplifica enormemente la prueba de cargas de archivos. Por ejemplo:

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
    

> {tip} Por defecto, el método `fake` eliminará todos los archivos en su directorio temporal. Si desea conservar estos archivos, puede usar a cambio el método "persistentFake".

<a name="mocking-facades"></a>

## Facades

A diferencia de las llamadas de métodos estáticos tradicionales, [facades](/docs/{{version}}/facades) puede ser simulado. Esto le proporciona una gran ventaja por sobre los métodos estáticos tradicionales y le ofrece la misma capacidad de prueba que tendría si estuviera usando la inyección de dependencia. Al realizar la prueba, puede con frecuencia querer simular una llamada a un facade Laravel en uno de sus controladores. Por ejemplo, considere la acción del siguiente controlador:

    <?php
    
    namespace App\Http\Controllers;
    
    use Illuminate\Support\Facades\Cache;
    
    class UserController extends Controller
    {
        /**
         * Show a list of all users of the application.
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');
    
            //
        }
    }
    

Podemos simular la llamada al facade `Cache` usando el método `shouldReceive`, el cual devolverá una instancia de una simulación [Mockery](https://github.com/padraic/mockery). Desde que facades en realidad se resuelve y maneja por el [service container](/docs/{{version}}/container) Laravel, tienen mucha más capacidad de prueba que una clase estática típica. Por ejemplo, simulemos nuestra llamada al método de facade `Cache` `get`:

    <?php
    
    namespace Tests\Feature;
    
    use Tests\TestCase;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    
    class UserControllerTest extends TestCase
    {
        public function testGetIndex()
        {
            Cache::shouldReceive('get')
                        ->once()
                        ->with('key')
                        ->andReturn('value');
    
            $response = $this->get('/users');
    
            // ...
        }
    }
    

> {note} No debería simular el facade `Request`. En cambio, pase la entrada que desee a los métodos de ayuda de HTTP, como `get` y `post` al ejecutar su prueba. Del mismo modo, en lugar de simular el facade `Config`, simplemente llame al método `Config::set` en sus pruebas.