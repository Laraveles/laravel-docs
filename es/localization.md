# Localization

- [Introducción](#introduction)
- [Definición de cadenas de traducción](#defining-translation-strings) 
    - [Utilizando claves – *short keys*](#using-short-keys)
    - [Usando cadenas de traducción como claves](#using-translation-strings-as-keys)
- [Definición de cadenas de traducción](#retrieving-translation-strings) 
    - [Reemplazar Parámetros en cadenas de traducción](#replacing-parameters-in-translation-strings)
    - [Pluralización](#pluralization)
- [Sobrescribir archivos de idioma de los paquetes](#overriding-package-language-files)

<a name="introduction"></a>

## Introducción

La característica de localización de Laravel provee una forma muy cómoda de obtener cadenas en varios lenguajes, permitiendo soportar múltiples idiomas en una aplicación de forma muy sencilla. Las cadenas de idioma se almacenan en archivos dentro del directorio `resources/lang`. Dentro de este directorio debería haber un subdirectorio para cada idioma soportado por la aplicación:

    /resources
        /lang
            /en
                messages.php
            /es
                messages.php
    

Todos los archivos de idioma únicamente retornan un array de cadenas con clave. Por ejemplo:

    <?php
    
    return [
        'welcome' => 'Welcome to our application'
    ];
    

### Configuración Regional

El idioma por defecto de la aplicación se almacena en el archivo de configuración `config/app.php`. Por supuesto, se puede modificar este valor para satisfacer las necesidades de la aplicación. Además se puede cambiar el idioma actual en tiempo de ejecución utilizando el método `setLocale` sobre la facade `App`:

    Route::get('welcome/{locale}', function ($locale) {
        App::setLocale($locale);
    
        //
    });
    

Puede configurar un "idioma de reserva", que se utilizará cuando el idioma activo no contenga una determinada cadena de traducción. Al igual que el idioma predeterminado, el lenguaje de reserva también se configura en el archivo `config/app. php`:

    'fallback_locale' => 'en',
    

#### Determinar el idioma actual

Puede utilizar los métodos `getLocale` y `isLocale` en la *facade* `App` para determinar la localización actual o comprobar si la localización es un valor dado:

    $locale = App::getLocale();
    
    if (App::isLocale('en')) {
        //
    }
    

<a name="defining-translation-strings"></a>

## Definición de cadenas de traducción

<a name="using-short-keys"></a>

### Utilizando claves – *short keys*

Normalmente, las cadenas de traducción se almacenan en archivos dentro del directorio `resources/lang`. Dentro de este directorio debería haber un subdirectorio para cada idioma soportado por la aplicación:

    /resources
        /lang
            /en
                messages.php
            /es
                messages.php
    

Todos los archivos de idioma únicamente retornan un *array* de cadenas con clave. Por ejemplo:

    <?php
    
    // resources/lang/en/messages.php
    
    return [
        'welcome' => 'Welcome to our application'
    ];
    

<a name="using-translation-strings-as-keys"></a>

### Usando cadenas de traducción como claves

Para aplicaciones con requisitos de traducción muy exigentes, definir cada cadena con una *"short key"* puede resultar muy confuso cuando se hace referencia a ellas en sus vistas. Por esta razón, Laravel también ofrece soporte para definir cadenas de traducción utilizando la traducción "por defecto" de la cadena como clave.

Los archivos de traducción que utilizan cadenas de traducción como claves se almacenan como archivos JSON en el directorio `resources/lang`. Por ejemplo, si su aplicación tiene una traducción al español, debe crear un archivo `resources/lang/es. json`:

    {
        "I love programming.": "Me encanta programar."
    }
    

<a name="retrieving-translation-strings"></a>

## Definición de cadenas de traducción

Puede recuperar líneas de los archivos de idioma utilizando el *helper* `__`. El método `__` acepta el archivo y la clave de la cadena de traducción como su primer argumento. Por ejemplo, para recuperar la cadena `welcome` del archivo `recursos/lang/messages.php`:

    echo __('messages.welcome');
    
    echo __('I love programming.');
    

Por supuesto, si está utilizando el motor de plantillas Blade, puede utilizar la sintaxis `{{ }}` para mostrar la cadena de traducción o utilizar la directiva `@lang`:

    {{ __('messages.welcome') }}
    
    @lang('messages.welcome')
    

Si la cadena de traducción especificada no existe, la función `__` devolverá la cadena de traducción para esa clave. Por lo tanto, utilizando el ejemplo anterior, la función `__` devolverá `messages.welcome` si la cadena de traducción no existe.

<a name="replacing-parameters-in-translation-strings"></a>

### Reemplazar parámetros en cadenas de traducción

Si se desea, se pueden establecer parámetros en las cadenas de traducción. Todos los parámetros contienen el prefijo `:`. Por ejemplo, se puede definir un mensaje de bienvenida con un nombre como parámetro:

    'welcome' => 'Welcome, :name',
    

To replace the place-holders when retrieving a translation string, pass an array of replacements as the second argument to the `__` function:

    echo __('messages.welcome', ['name' => 'dayle']);
    

If your place-holder contains all capital letters, or only has its first letter capitalized, the translated value will be capitalized accordingly:

    'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
    'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle
    

<a name="pluralization"></a>

### Pluralization

Pluralization is a complex problem, as different languages have a variety of complex rules for pluralization. By using a "pipe" character, you may distinguish singular and plural forms of a string:

    'apples' => 'There is one apple|There are many apples',
    

You may even create more complex pluralization rules which specify translation strings for multiple number ranges:

    'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',
    

After defining a translation string that has pluralization options, you may use the `trans_choice` function to retrieve the line for a given "count". In this example, since the count is greater than one, the plural form of the translation string is returned:

    echo trans_choice('messages.apples', 10);
    

<a name="overriding-package-language-files"></a>

## Overriding Package Language Files

Some packages may ship with their own language files. Instead of changing the package's core files to tweak these lines, you may override them by placing files in the `resources/lang/vendor/{package}/{locale}` directory.

So, for example, if you need to override the English translation strings in `messages.php` for a package named `skyrim/hearthfire`, you should place a language file at: `resources/lang/vendor/hearthfire/en/messages.php`. Within this file, you should only define the translation strings you wish to override. Any translation strings you don't override will still be loaded from the package's original language files.