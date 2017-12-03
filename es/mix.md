# Compilando *Assets* (Laravel Mix)

- [Introducción](#introduction)
- [Instalación & configuración](#installation)
- [Ejecutar Mix](#running-mix)
- [Trabajar con hojas de estilo](#working-with-stylesheets) 
    - [Less](#less)
    - [Sass](#sass)
    - [Stylus](#stylus)
    - [PostCSS](#postcss)
    - [CSS simple](#plain-css)
    - [Procesamiento de URL](#url-processing)
    - [Mapas de fuentes – *Source maps*](#css-source-maps)
- [Trabajar con JavaScript](#working-with-scripts) 
    - [Extracción del *Vendor*](#vendor-extraction)
    - [React](#react)
    - [Vanilla JS](#vanilla-js)
    - [Configuración personalizada de Webpack](#custom-webpack-configuration)
- [Copiar archivos & directorios](#copying-files-and-directories)
- [Versionado/evitar caché](#versioning-and-cache-busting)
- [Recarga Browsersync](#browsersync-reloading)
- [Variables de entorno](#environment-variables)
- [Notificaciones](#notifications)

<a name="introduction"></a>

## Introducción

[Laravel Mix](https://github.com/JeffreyWay/laravel-mix) proporciona una API fluida para definir los pasos de compilación de Webpack para su aplicación Laravel utilizando varios preprocesadores comunes de CSS y JavaScript. Encadenando métodos, puede definir con fluidez su *asset pipeline*. Por ejemplo:

    mix.js('resources/assets/js/app.js', 'public/js')
       .sass('resources/assets/sass/app.scss', 'public/css');
    

Si alguna vez ha estado confundido y abrumado sobre cómo comenzar con la compilación de *assets* y Webpack, le encantará Laravel Mix. Sin embargo, no es necesario utilizarlo para el desarrollo de la aplicación. Por supuesto, puede usar cualquier herramienta para el tratamiento de *assets* que desee, o incluso ninguna.

<a name="installation"></a>

## Instalación & configuración

#### Instalar Node

Antes de activar Mix, primero debe asegurarse de que Node.js y NPM estén instalados en su máquina.

    node -v
    npm -v
    

Por defecto, Laravel Homestead incluye todo lo que necesita; sin embargo, si no está utilizando Vagrant, puede instalar fácilmente la última versión de Node y NPM usando instaladores gráficos simples desde [su p&aacute;gina de descarga](https://nodejs.org/en/download/).

#### Laravel Mix

El único paso restante es instalar Laravel Mix. Dentro de una nueva instalación de Laravel, encontrará un archivo `package.json` en la raíz de la estructura de su directorio. El archivo predeterminado `package.json` incluye todo lo que necesita para comenzar. Este archivo es similar a `composer.json`, excepto que define las dependencias de Node en lugar de PHP. Se pueden instalar estas dependencias ejecutando:

    npm install
    

<a name="running-mix"></a>

## Ejecutar Mix

Mix es una capa de configuración por encima de [Webpack](https://webpack.js.org), por lo que para ejecutar las tareas de Mix solo es necesario ejecutar uno de los scripts de NPM que se incluye con el archivo Laravel predeterminado `package.json`:

    // Run all Mix tasks...
    npm run dev
    
    // Run all Mix tasks and minify output...
    npm run production
    

#### Observar cambios en *assets*

El comando `npm run watch` continuará ejecutándose en su terminal y observará todos los ficheros relevantes en busca de cambios. Webpack luego recompilará automáticamente estos *assets* cuando detecte un cambio:

    npm run watch
    

Puede encontrar que en ciertos entornos Webpack no se actualiza cuando se modifican sus archivos. Si este es el caso en su sistema, considerar el comando `watch-poll`:

    npm run watch-poll
    

<a name="working-with-stylesheets"></a>

## Trabajar con hojas de estilo

El archivo `webpack.mix.js` es el punto de entrada para la compilación de todos los *assets*. Piense en ello como un contenedor de configuración ligera de Webpack. Las tareas Mix pueden encadenarse juntas para definir exactamente cómo deben compilarse los *assets*.

<a name="less"></a>

### Less

El método `Less` se puede usar para compilar [Less](http://lesscss.org/) en CSS. Vamos a compilar nuestro archivo principal `app.less` a `public/css/app.css`.

    mix.less('resources/assets/less/app.less', 'public/css');
    

Se pueden realizar varias llamadas al método `less` para compilar varios archivos:

    mix.less('resources/assets/less/app.less', 'public/css')
       .less('resources/assets/less/admin.less', 'public/css');
    

Si desea personalizar el nombre del archivo compilado CSS, puede pasar una ruta completa del archivo como el segundo argumento para el método `less`:

    mix.less('resources/assets/less/app.less', 'public/stylesheets/styles.css');
    

Si necesita anular las [opciones del plug-in Less subyacentes](https://github.com/webpack-contrib/less-loader#options), puede pasar un objeto como tercer argumento para `mix.less()`:

    mix.less('resources/assets/less/app.less', 'public/css', {
        strictMath: true
    });
    

<a name="sass"></a>

### Sass

El método `sass` te permite compilar [Sass](http://sass-lang.com/) en CSS. Puede usar el método de esta manera:

    mix.sass('resources/assets/sass/app.scss', 'public/css');
    

De nuevo, al igual que el método `less`, puede compilar varios archivos Sass en sus respectivos archivos CSS e incluso personalizar el directorio de salida del CSS resultante:

    mix.sass('resources/assets/sass/app.sass', 'public/css')
       .sass('resources/assets/sass/admin.sass', 'public/css/admin');
    

Se pueden proporcionar [opciones del plug-in Node-Sass](https://github.com/sass/node-sass#options) como tercer argumento:

    mix.sass('resources/assets/sass/app.sass', 'public/css', {
        precision: 5
    });
    

<a name="stylus"></a>

### Stylus

Similar a Less y Sass, el método `stylus` le permite compilar [Stylus](http://stylus-lang.com/) en CSS:

    mix.stylus('resources/assets/stylus/app.styl', 'public/css');
    

También puede instalar complementos adicionales de Stylus, como [Rupture](https://github.com/jescalan/rupture). Primero, instale el complemento en cuestión a través de NPM (`npm install rupture`) y luego lo requiere en su llamada a `mix.stylus()`:

    mix.stylus('resources/assets/stylus/app.styl', 'public/css', {
        use: [
            require('rupture')()
        ]
    });
    

<a name="postcss"></a>

### PostCSS

[PostCSS](http://postcss.org/), una herramienta poderosa para transformar su CSS, se incluye con Laravel Mix de serie. De forma predeterminada, Mix aprovecha el popular *plug-in* [Autoprefixer](https://github.com/postcss/autoprefixer) para aplicar automáticamente todos los prefijos de CSS necesarios. Sin embargo, puede agregar complementos adicionales que sean apropiados para su aplicación. Primero, instale el complemento deseado a través de NPM y luego hágalo en su archivo `webpack.mix.js`:

    mix.sass('resources/assets/sass/app.scss', 'public/css')
       .options({
            postCss: [
                require('postcss-css-variables')()
            ]
       });
    

<a name="plain-css"></a>

### CSS simple

Si simplemente desea concatenar algunas hojas de estilos CSS simples en un solo archivo, puede usar el método `styles`.

    mix.styles([
        'public/css/vendor/normalize.css',
        'public/css/vendor/videojs.css'
    ], 'public/css/all.css');
    

<a name="url-processing"></a>

### Procesamiento URL

Debido a que Laravel Mix se basa en Webpack, es importante comprender algunos conceptos de Webpack. Para la compilación de CSS, Webpack reescribirá y optimizará cualquier llamada `url()` dentro de sus hojas de estilo. Si bien esto inicialmente puede sonar extraño, es una funcionalidad increíblemente poderosa. Imagine que queremos compilar Sass que incluye una URL relativa a una imagen:

    .example {
        background: url('../images/example.png');
    }
    

> {note} Las rutas absolutas para cualquier `url()` determinada se excluirán de la reescritura de URL. Por ejemplo, `url('/images/thing.png')` o `url('http://example.com/images/thing.png')` no será modificado.

De forma predeterminada, Laravel Mix y Webpack encontrarán `example.png`, lo copiarán en su carpeta `public/images` y luego reescribirá la `url()` dentro de su hoja de estilo generada. Como tal, su CSS compilado será:

    .example {
      background: url(/images/example.png?d41d8cd98f00b204e9800998ecf8427e);
    }
    

Tan útil como puede ser esta característica, es posible que su estructura de carpetas existente ya esté configurada de la manera que desee. Si este es el caso, puede deshabilitar la reescritura de `url()` así:

    mix.sass('resources/assets/app/app.scss', 'public/css')
       .options({
          processCssUrls: false
       });
    

Con esta adición a su archivo `webpack.mix.js`, Mix ya no coincidirá con ninguna `url()` o copiará los *assets* en su directorio *public*. En otras palabras, el CSS compilado se verá exactamente como se escribió originalmente:

    .example {
        background: url("../images/thing.png");
    }
    

<a name="css-source-maps"></a>

### Mapas de fuentes – *Source maps*

Aunque está deshabilitado de forma predeterminada, los *source maps* se pueden activar llamando al método `mix.sourceMaps()` en su archivo `webpack.mix.js`. A pesar de que viene con un costo de compilación/rendimiento, esto proporcionará información de depuración adicional a las herramientas de desarrollo de su navegador al usar recursos compilados.

    mix.js('resources/assets/js/app.js', 'public/js')
       .sourceMaps();
    

<a name="working-with-scripts"></a>

## Trabajar con JavaScript

Mix proporciona varias funciones para ayudar a trabajar con sus archivos JavaScript, como la compilación de ECMAScript 2015, la agrupación de módulos, la minificación y simplemente la concatenación de archivos JavaScript simples. Aún mejor, todo esto funciona a la perfección, sin requerir una pizca de configuración personalizada:

    mix.js('resources/assets/js/app.js', 'public/js');
    

Con esta única línea de código, ahora puede aprovechar:

<div class="content-list">
  <ul>
    <li>
      Sintaxis ES2015.
    </li>
    <li>
      Módulos
    </li>
    <li>
      Compilación de archivos <code>.vue</code>.
    </li>
    <li>
      Minificación para entornos de producción.
    </li>
  </ul>
</div>

<a name="vendor-extraction"></a>

### Extracción del *Vendor*

Una posible desventaja de agrupar todo el JavaScript específico de la aplicación con las bibliotecas de su *vendor* es que hace que el almacenamiento en caché a largo plazo sea más difícil. Por ejemplo, una sola actualización de su código de aplicación obligará al navegador a volver a descargar todas las bibliotecas de su *vendor*, incluso si no han cambiado.

Si tiene la intención de realizar actualizaciones frecuentes del JavaScript de su aplicación, debería considerar extraer todas sus librerías del *vendor* en su propio archivo. De esta forma, un cambio en el código de su aplicación no afectará el almacenamiento en caché de su gran archivo `vendor.js`. El método `exctract` de Mix hace que esto sea muy sencillo:

    mix.js('resources/assets/js/app.js', 'public/js')
       .extract(['vue'])
    

El método `extract` acepta una matriz de todas las librerías o módulos que desea extraer en un archivo `vendor.js`. Usando el fragmento de arriba como ejemplo, Mix generará los siguientes archivos:

<div class="content-list">
  <ul>
    <li>
      <code>public/js/manifest.js</code>: <em>El tiempo de ejecución del manifiesto de Webpack</em>
    </li>
    <li>
      <code>public/js/vendor.js</code>: <em>Sus librerías del vendor</em>
    </li>
    <li>
      <code>public/js/app.js</code>: <em>Su código de aplicación</em>
    </li>
  </ul>
</div>

Para evitar errores de JavaScript, asegúrese de cargar estos archivos en el orden correcto:

    <script src="/js/manifest.js"></script>
    <script src="/js/vendor.js"></script>
    <script src="/js/app.js"></script>
    

<a name="react"></a>

### React

Mix puede instalar automáticamente los *plug-ins* de Babel necesarios para la compatibilidad con React. Para comenzar, reemplace su llamada a `mix.js()` con `mix.react()`:

    mix.react('resources/assets/js/app.jsx', 'public/js');
    

Mix descargará e incluirá el *plug-in* de Babel `babel-preset-react`.

<a name="vanilla-js"></a>

### Vanilla JS

Similar a la combinación de hojas de estilo con `mix.styles()`, también puedes combinar y minificar cualquier cantidad de archivos JavaScript con el método `scripts()`:

    mix.scripts([
        'public/js/admin.js',
        'public/js/dashboard.js'
    ], 'public/js/all.js');
    

Esta opción es particularmente útil para proyectos antiguos donde no se requiere compilación de Webpack para su JavaScript.

> {tip} Una ligera variación de `mix.scripts()` es `mix.babel()`. Su firma de método es idéntica a `scripts`; sin embargo, el archivo concatenado recibirá la compilación de Babel, que traduce cualquier código de ES2015 a *vanilla JavaScript* que todos los navegadores entenderán.

<a name="custom-webpack-configuration"></a>

### Configuración personalizada de Webpack

Entre bastidores, Laravel Mix hace referencia a un archivo `webpack.config.js` pre-configurado para que pueda comenzar a trabajar lo más rápido posible. En ocasiones, puede necesitar modificar manualmente este archivo. Es posible que tenga un cargador o *plug-in* especial al que se deba hacer referencia, o tal vez prefiera usar Stylus en lugar de Sass. En tales casos, tiene dos opciones:

#### Fusionar la configuración personalizada

Mix proporciona un método útil `webpackConfig` que le permite combinar cualquier anulación de configuración de Webpack. Esta es una opción particularmente atractiva, ya que no requiere que copie y mantenga su propia copia del archivo `webpack.config.js`. El método `webpackConfig` acepta un objeto, que debe contener cualquier [configuración específica de Webpack](https://webpack.js.org/configuration/) que desee aplicar.

    mix.webpackConfig({
        resolve: {
            modules: [
                path.resolve(__dirname, 'vendor/laravel/spark/resources/assets/js')
            ]
        }
    });
    

#### Archivos de configuración personalizados

Si desea personalizar completamente la configuración de su Webpack, copie el archivo `node_modules/laravel-mix/setup/webpack.config.js` en el directorio raíz de su proyecto. A continuación, apunte todas las referencias de `--config` en su archivo `package.json` al archivo de configuración recién copiado. Si opta por llevar este enfoque, cualquier actualización futura de del archivo `webpack.config.js` de Mix deberá fusionarse manualmente en su archivo personalizado.

<a name="copying-files-and-directories"></a>

## Copiar archivos & directorios

El método `copy` se puede usar para copiar archivos y directorios a ubicaciones nuevas. Puede ser útil cuando un recurso particular dentro de su directorio `node_modules` necesita ser reubicado en su carpeta `public`.

    mix.copy('node_modules/foo/bar.css', 'public/css/bar.css');
    

Al copiar un directorio, el método `copy` aplanará la estructura del directorio. Para mantener la estructura original del directorio, debe usar el método `copyDirectory` en su lugar:

    mix.copyDirectory('assets/img', 'public/img');
    

<a name="versioning-and-cache-busting"></a>

## Versionado/evitar caché

Muchos desarrolladores establecen un *timestamp* o *token* único como sufijo para sus archivos compilados para forzar a los navegadores cargar las nuevas copias en lugar de servir copias obsoletas del código. Mix gestiona esto utilizando el método `version`.

El método `version` agregará automáticamente un *hash* único a los nombres de archivo de todos los archivos compilados, lo que permite un almacenamiento en memoria caché más conveniente:

    mix.js('resources/assets/js/app.js', 'public/js')
       .version();
    

Después de generar el archivo versionado, no sabrá el nombre exacto del archivo. Por lo tanto, debe usar la función global de Laravel `mix` en sus [vistas](/docs/{{version}}/views) para cargar el *asset* adecuado. La función `mix` determinará automáticamente el nombre actual del archivo:

    <link rel="stylesheet" href="{{ mix('/css/app.css') }}">
    

Debido a que los archivos versionados generalmente no son necesarios en el desarrollo, puede indicarle al proceso de versiones que solo se ejecute durante `npm run production`:

    mix.js('resources/assets/js/app.js', 'public/js');
    
    if (mix.inProduction()) {
        mix.version();
    }
    

<a name="browsersync-reloading"></a>

## Recarga Browsersync

[BrowserSync](https://browsersync.io/) puede supervisar automáticamente sus archivos en busca de cambios e inyectar sus cambios en el navegador sin necesidad de una actualización manual. Puede habilitar el soporte llamando al método `mix.browserSync()`:

    mix.browserSync('my-domain.dev');
    
    // Or...
    
    // https://browsersync.io/docs/options
    mix.browserSync({
        proxy: 'my-domain.dev'
    });
    

Puede pasar una cadena (proxy) u objeto (configuración BrowserSync) a este método. A continuación, inicie el servidor de desarrollo de Webpack con el comando `npm run watch`. Ahora, cuando modifique un script o un archivo PHP, observe cómo el navegador actualiza al instante la página para reflejar sus cambios.

<a name="environment-variables"></a>

## Variables de entorno

Puede inyectar variables de entorno en Mix prefijando una clave en su archivo `.env` con `MIX_`:

    MIX_SENTRY_DSN_PUBLIC=http://example.com
    

Después de que la variable se haya definido en su archivo `.env`, puede acceder a través del objeto `process.env`. Si el valor cambia mientras está ejecutando una tarea `watch`, deberá reiniciar la tarea:

    process.env.MIX_SENTRY_DSN_PUBLIC
    

<a name="notifications"></a>

## Notificaciones

Cuando esté disponible, Mix mostrará automáticamente las notificaciones del sistema operativo para cada paquete. Esto le dará una respuesta instantánea, si la compilación fue exitosa o no. Sin embargo, puede haber instancias en las que prefiera inhabilitar estas notificaciones. Uno de esos ejemplos podría ser activar Mix en su servidor de producción. Las notificaciones se pueden desactivar mediante el método `disableNotifications`.

    mix.disableNotifications();