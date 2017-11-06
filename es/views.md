# Vistas

- [Crear vistas](#creating-views)
- [Pasar datos a vistas](#passing-data-to-views) 
    - [Compartir datos en todas las vistas](#sharing-data-with-all-views)
- [*Composers* de Vistas (compositores)](#view-composers)

<a name="creating-views"></a>

## Crear Vistas

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

As you saw in the previous examples, you may pass an array of data to views:

    return view('greetings', ['name' => 'Victoria']);
    

When passing information in this manner, the data should be an array with key / value pairs. Inside your view, you can then access each value using its corresponding key, such as `<?php echo $key; ?>`. As an alternative to passing a complete array of data to the `view` helper function, you may use the `with` method to add individual pieces of data to the view:

    return view('greeting')->with('name', 'Victoria');
    

<a name="sharing-data-with-all-views"></a>

#### Sharing Data With All Views

Occasionally, you may need to share a piece of data with all views that are rendered by your application. You may do so using the view facade's `share` method. Typically, you should place calls to `share` within a service provider's `boot` method. You are free to add them to the `AppServiceProvider` or generate a separate service provider to house them:

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

## View Composers

Los View composers son callbacks o métodos de una clase que son llamados cuando una vista es renderizada. If you have data that you want to be bound to a view each time that view is rendered, a view composer can help you organize that logic into a single location.

For this example, let's register the view composers within a [service provider](/docs/{{version}}/providers). We'll use the `View` facade to access the underlying `Illuminate\Contracts\View\Factory` contract implementation. Remember, Laravel does not include a default directory for view composers. You are free to organize them however you wish. For example, you could create an `app/Http/ViewComposers` directory:

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
    

> {note} Remember, if you create a new service provider to contain your view composer registrations, you will need to add the service provider to the `providers` array in the `config/app.php` configuration file.

Now that we have registered the composer, the `ProfileComposer@compose` method will be executed each time the `profile` view is being rendered. So, let's define the composer class:

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
    

Just before the view is rendered, the composer's `compose` method is called with the `Illuminate\View\View` instance. You may use the `with` method to bind data to the view.

> {tip} All view composers are resolved via the [service container](/docs/{{version}}/container), so you may type-hint any dependencies you need within a composer's constructor.

#### Attaching A Composer To Multiple Views

You may attach a view composer to multiple views at once by passing an array of views as the first argument to the `composer` method:

    View::composer(
        ['profile', 'dashboard'],
        'App\Http\ViewComposers\MyViewComposer'
    );
    

The `composer` method also accepts the `*` character as a wildcard, allowing you to attach a composer to all views:

    View::composer('*', function ($view) {
        //
    });
    

#### View Creators

View **creators** are very similar to view composers; however, they are executed immediately after the view is instantiated instead of waiting until the view is about to render. To register a view creator, use the `creator` method:

    View::creator('profile', 'App\Http\ViewCreators\ProfileCreator');