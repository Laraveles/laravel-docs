# Vistas

- [Crear vistas](#creating-views)
- [Pasar datos a vistas](#passing-data-to-views) 
    - [Compartir datos en todas las vistas](#sharing-data-with-all-views)
- [*Composers* de Vistas (compositores)](#view-composers)

<a name="creating-views"></a>

## Crear vistas

> {tip} Para saber más sobre como escribir plantillas en Blade, comprobar la documentación completa de [Blade](/docs/{{version}}/blade).

Las vistas contienen el HTML que se sirve por cualquier aplicación y separa la lógica del controlador/aplicación de la lógica de presentación. Las vistas se almacenan en la carpeta `resources/views`. Una vista simple podría ser algo como esto:

    <!-- View stored in resources/views/greeting.blade.php -->
    
    <html>
        <body>
            <h1>Hello, {{ $name }}</h1>
        </body>
    </html>
    

Puesto que esta vista se almacena en `resources/views/greetings.blade.php`, se retornará utilizando la función global `view` de este modo:

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });
    

Como se puede observar, el primer argumento pasado al *helper* `view` corresponde al nombre del archivo ubicado en el directorio `resources/views`. El segundo parámetro es un array de datos que debe estar presente en la vista. En este caso, se le está pasando la variable `name`, la cual se muestra en la vista utilizando la [sintaxis Blade](/docs/{{version}}/blade).

Por su puesto, las vistas pueden estar anidadas dentro de cualquier sub-directorio de la carpeta `resources/views`. La notación de "puntos" o "*dot notation*" se utiliza para hacer referencia a vistas anidadas. Por ejemplo, si una vista está almacenada en `resources/views/admin/profile.blade.php`, se puede hacer referencia a ella de la siguiente manera:

    return view('admin.profile', $data);
    

#### Determinar si existe una vista

Para determinar si una vista existe, se puede utilizar la facade `View`. El método `exists` retornará `true` si la vista existe:

    use Illuminate\Support\Facades\View;
    
    if (View::exists('emails.customer')) {
        //
    }
    

#### Crear la primera vista disponible

Utilizando el método `first` se puede crear la primera vista que se encuentre en un array de vistas. Es útil si una aplicación o paquete permite personalizar o sobrescribir las vistas:

    return view()->first(['custom.admin', 'admin'], $data);
    

Por supuesto, también se puede llamar a este método a través de la [facade](/docs/{{version}}/facades) `View`:

    use Illuminate\Support\Facades\View;
    
    return View::first(['custom.admin', 'admin'], $data);
    

<a name="passing-data-to-views"></a>

## Pasar datos a vistas

Como se ha visto en ejemplos anteriores, se puede pasar un array de datos a las vistas:

    return view('greetings', ['name' => 'Victoria']);
    

Al pasar información de esta forma, los datos deben ser un array con pares de clave/valor. Dentro de la vista, se pueden acceder a estos valores utilizando su clave correspondiente, tal como `<?php echo $key; ?>`. Como alternativa a pasar un array de datos al helper `view`, se pueden pasar piezas individuales de datos a la vista utilizando el método `with`:

    return view('greeting')->with('name', 'Victoria');
    

<a name="sharing-data-with-all-views"></a>

#### Compartir datos en todas las vistas

Ocasionalmente, puede surgir la necesidad de compartir ciertos datos en todas las vistas de la aplicación. Esto se puede hacer utilizando el método de la facade `share`. Por lo general, las llamadas al método `share` se hacen dentro del método `boot` de un service provider. Estas llamadas se pueden añadir en `AppServiceProvider` o generar un service provider independiente para alojarlas:

    <?php
    
    namespace App\Providers;
    
    use Illuminate\Support\Facades\View;
    
    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            View::share('key', 'value');
        }
    
        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }
    

<a name="view-composers"></a>

## *Composers* de Vistas (compositores)

Los *view composers* son *callbacks* o métodos de una clase que son llamados cuando una vista es renderizada. Si hay datos susceptibles de ser vinculados a una vista cada vez que esta es renderizada, un *view composer* ayudará a organizar esta lógica en una única ubicación.

Para este ejemplo se registrarán los *view composer* en un [service provider](/docs/{{version}}/providers). Se utilizará la *facade* `View` para acceder a la implementación subyacente del contrato `Illuminate\Contracts\View\Factory`. Laravel no incluye un directorio predeterminado para los *view composer*. Así que se deja a libertad el programador de organizarlos como desee. Por ejemplo, se podría crear un directorio `app/Http/ViewComposers`:

    <?php
    
    namespace App\Providers;
    
    use Illuminate\Support\Facades\View;
    use Illuminate\Support\ServiceProvider;
    
    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function boot()
        {
            // Using class based composers...
            View::composer(
                'profile', 'App\Http\ViewComposers\ProfileComposer'
            );
    
            // Using Closure based composers...
            View::composer('dashboard', function ($view) {
                //
            });
        }
    
        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }
    

> {note} Si se crea un nuevo *service provider* que almacene los registros de *view composers*, será necesario añadirlo al array de `providers` en el archivo de configuración `config/app.php`.

Luego de registrar el *composer*, el método `ProfileComposer@compose` se ejecutara cada vez que la vista `profile` esté renderizada. Por lo tanto, se define la clase del *composer*:

    <?php
    
    namespace App\Http\ViewComposers;
    
    use Illuminate\View\View;
    use App\Repositories\UserRepository;
    
    class ProfileComposer
    {
        /**
         * The user repository implementation.
         *
         * @var UserRepository
         */
        protected $users;
    
        /**
         * Create a new profile composer.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            // Dependencies automatically resolved by service container...
            $this->users = $users;
        }
    
        /**
         * Bind data to the view.
         *
         * @param  View  $view
         * @return void
         */
        public function compose(View $view)
        {
            $view->with('count', $this->users->count());
        }
    }
    

Justo antes de que la vista sea renderizada, los *composers* del metodo `compose` se llaman desde la instancia `Illuminate\Contracts\View\View`. Se puede utilizar el método `with` para enlazar datos con la vista.

> {tip} Todos los *view composers* se resuelven a través del [service container](/docs/{{version}}/container), así que se pueden añadir tantas dependencias como se necesiten en el constructor del *composer*.

#### Asociar un *composer* a varias vistas

Se puede conectar un view composer a múltiples vistas al mismo tiempo pasando un array de las vistas como primer argumento del método `composer`:

    View::composer(
        ['profile', 'dashboard'],
        'App\Http\ViewComposers\MyViewComposer'
    );
    

El método `composer` acepta el carácter `*` como comodín, permitiendo adjuntar un *composer* a todas las vistas:

    View::composer('*', function ($view) {
        //
    });
    

#### *View creators</en></h4> 

Los View **creators** son muy similares a los *view composers*; sin embargo se ejecutan inmediatamente después de que una vista se instancie en lugar de esperar a que esté a punto de renderizar. Para registrar un *view creator*, simplemente se utiliza el método `creator`:

    View::creator('profile', 'App\Http\ViewCreators\ProfileCreator');