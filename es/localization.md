# Localización – *Locale*

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
    

Para reemplazar los marcadores de posición cuando se recupera una cadena de traducción, pase una matriz de reemplazos como segundo argumento a la función `__`:

    echo __('messages.welcome', ['name' => 'dayle']);
    

Si su parámetro contiene todas las letras mayúsculas, o sólo tiene su primera letra en mayúscula, el valor traducido será en mayúsculas y minúsculas:

    'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
    'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle
    

<a name="pluralization"></a>

### Pluralización

La pluralización es un problema complejo, pues diferentes idiomas tienen diferentes reglas de pluralización. Utilizando el caracter "pipe" (tubería o barra vertical), se puede distinguir entre la forma singular y plural de una cadena:

    'apples' => 'There is one apple|There are many apples',
    

Incluso puede crear reglas de pluralización más complejas que especifiquen cadenas de traducción para múltiples rangos de números:

    'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',
    

Después de definir una cadena de traducción que tenga opciones de pluralización, puede utilizar la función `trans_choice` para recuperar la línea para un "conteo" determinado. En este ejemplo, puesto que el contador es mayor que uno, se retornará la forma plural:

    echo trans_choice('messages.apples', 10);
    

<a name="overriding-package-language-files"></a>

## Sobrescribir archivos de idioma de los paquetes

Algunos paquetes incluyen sus propios archivos de idioma. En lugar de cambiar los archivos centrales del paquete para ajustar estas líneas, puede anularlas colocando archivos en el directorio `resources/lang/vendor/{package}/{locale}`.

Así, por ejemplo, si necesita anular las cadenas de traducción al inglés en `messages.php` para un paquete llamado `skyrim/hearthfire`, debería colocar un archivo de idioma en: `resources/lang/vendor/hearthfire/en/messages.php`. Dentro de este fichero, sólo debe definir las cadenas de traducción que desea anular. Cualquier cadena de traducción que no sobreescriba se cargará desde los archivos de idioma originales del paquete.