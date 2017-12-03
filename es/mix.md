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

[PostCSS](http://postcss.org/), una herramienta poderosa para transformar su CSS, se incluye con Laravel Mix de serie. By default, Mix leverages the popular [Autoprefixer](https://github.com/postcss/autoprefixer) plug-in to automatically apply all necessary CSS3 vendor prefixes. However, you're free to add any additional plug-ins that are appropriate for your application. First, install the desired plug-in through NPM and then reference it in your `webpack.mix.js` file:

    mix.sass('resources/assets/sass/app.scss', 'public/css')
       .options({
            postCss: [
                require('postcss-css-variables')()
            ]
       });
    

<a name="plain-css"></a>

### Plain CSS

If you would just like to concatenate some plain CSS stylesheets into a single file, you may use the `styles` method.

    mix.styles([
        'public/css/vendor/normalize.css',
        'public/css/vendor/videojs.css'
    ], 'public/css/all.css');
    

<a name="url-processing"></a>

### URL Processing

Because Laravel Mix is built on top of Webpack, it's important to understand a few Webpack concepts. For CSS compilation, Webpack will rewrite and optimize any `url()` calls within your stylesheets. While this might initially sound strange, it's an incredibly powerful piece of functionality. Imagine that we want to compile Sass that includes a relative URL to an image:

    .example {
        background: url('../images/example.png');
    }
    

> {note} Absolute paths for any given `url()` will be excluded from URL-rewriting. For example, `url('/images/thing.png')` or `url('http://example.com/images/thing.png')` won't be modified.

By default, Laravel Mix and Webpack will find `example.png`, copy it to your `public/images` folder, and then rewrite the `url()` within your generated stylesheet. As such, your compiled CSS will be:

    .example {
      background: url(/images/example.png?d41d8cd98f00b204e9800998ecf8427e);
    }
    

As useful as this feature may be, it's possible that your existing folder structure is already configured in a way you like. If this is the case, you may disable `url()` rewriting like so:

    mix.sass('resources/assets/app/app.scss', 'public/css')
       .options({
          processCssUrls: false
       });
    

With this addition to your `webpack.mix.js` file, Mix will no longer match any `url()` or copy assets to your public directory. In other words, the compiled CSS will look just like how you originally typed it:

    .example {
        background: url("../images/thing.png");
    }
    

<a name="css-source-maps"></a>

### Source Maps

Though disabled by default, source maps may be activated by calling the `mix.sourceMaps()` method in your `webpack.mix.js` file. Though it comes with a compile/performance cost, this will provide extra debugging information to your browser's developer tools when using compiled assets.

    mix.js('resources/assets/js/app.js', 'public/js')
       .sourceMaps();
    

<a name="working-with-scripts"></a>

## Working With JavaScript

Mix provides several features to help you work with your JavaScript files, such as compiling ECMAScript 2015, module bundling, minification, and simply concatenating plain JavaScript files. Even better, this all works seamlessly, without requiring an ounce of custom configuration:

    mix.js('resources/assets/js/app.js', 'public/js');
    

With this single line of code, you may now take advantage of:

<div class="content-list">
  <ul>
    <li>
      ES2015 syntax.
    </li>
    <li>
      Modules
    </li>
    <li>
      Compilation of <code>.vue</code> files.
    </li>
    <li>
      Minification for production environments.
    </li>
  </ul>
</div>

<a name="vendor-extraction"></a>

### Vendor Extraction

One potential downside to bundling all application-specific JavaScript with your vendor libraries is that it makes long-term caching more difficult. For example, a single update to your application code will force the browser to re-download all of your vendor libraries even if they haven't changed.

If you intend to make frequent updates to your application's JavaScript, you should consider extracting all of your vendor libraries into their own file. This way, a change to your application code will not affect the caching of your large `vendor.js` file. Mix's `extract` method makes this a breeze:

    mix.js('resources/assets/js/app.js', 'public/js')
       .extract(['vue'])
    

The `extract` method accepts an array of all libraries or modules that you wish to extract into a `vendor.js` file. Using the above snippet as an example, Mix will generate the following files:

<div class="content-list">
  <ul>
    <li>
      <code>public/js/manifest.js</code>: <em>The Webpack manifest runtime</em>
    </li>
    <li>
      <code>public/js/vendor.js</code>: <em>Your vendor libraries</em>
    </li>
    <li>
      <code>public/js/app.js</code>: <em>Your application code</em>
    </li>
  </ul>
</div>

To avoid JavaScript errors, be sure to load these files in the proper order:

    <script src="/js/manifest.js"></script>
    <script src="/js/vendor.js"></script>
    <script src="/js/app.js"></script>
    

<a name="react"></a>

### React

Mix can automatically install the Babel plug-ins necessary for React support. To get started, replace your `mix.js()` call with `mix.react()`:

    mix.react('resources/assets/js/app.jsx', 'public/js');
    

Behind the scenes, Mix will download and include the appropriate `babel-preset-react` Babel plug-in.

<a name="vanilla-js"></a>

### Vanilla JS

Similar to combining stylesheets with `mix.styles()`, you may also combine and minify any number of JavaScript files with the `scripts()` method:

    mix.scripts([
        'public/js/admin.js',
        'public/js/dashboard.js'
    ], 'public/js/all.js');
    

This option is particularly useful for legacy projects where you don't require Webpack compilation for your JavaScript.

> {tip} A slight variation of `mix.scripts()` is `mix.babel()`. Its method signature is identical to `scripts`; however, the concatenated file will receive Babel compilation, which translates any ES2015 code to vanilla JavaScript that all browsers will understand.

<a name="custom-webpack-configuration"></a>

### Custom Webpack Configuration

Behind the scenes, Laravel Mix references a pre-configured `webpack.config.js` file to get you up and running as quickly as possible. Occasionally, you may need to manually modify this file. You might have a special loader or plug-in that needs to be referenced, or maybe you prefer to use Stylus instead of Sass. In such instances, you have two choices:

#### Merging Custom Configuration

Mix provides a useful `webpackConfig` method that allows you to merge any short Webpack configuration overrides. This is a particularly appealing choice, as it doesn't require you to copy and maintain your own copy of the `webpack.config.js` file. The `webpackConfig` method accepts an object, which should contain any [Webpack-specific configuration](https://webpack.js.org/configuration/) that you wish to apply.

    mix.webpackConfig({
        resolve: {
            modules: [
                path.resolve(__dirname, 'vendor/laravel/spark/resources/assets/js')
            ]
        }
    });
    

#### Custom Configuration Files

If you would like completely customize your Webpack configuration, copy the `node_modules/laravel-mix/setup/webpack.config.js` file to your project's root directory. Next, point all of the `--config` references in your `package.json` file to the newly copied configuration file. If you choose to take this approach to customization, any future upstream updates to Mix's `webpack.config.js` must be manually merged into your customized file.

<a name="copying-files-and-directories"></a>

## Copying Files & Directories

The `copy` method may be used to copy files and directories to new locations. This can be useful when a particular asset within your `node_modules` directory needs to be relocated to your `public` folder.

    mix.copy('node_modules/foo/bar.css', 'public/css/bar.css');
    

When copying a directory, the `copy` method will flatten the directory's structure. To maintain the directory's original structure, you should use the `copyDirectory` method instead:

    mix.copyDirectory('assets/img', 'public/img');
    

<a name="versioning-and-cache-busting"></a>

## Versioning / Cache Busting

Many developers suffix their compiled assets with a timestamp or unique token to force browsers to load the fresh assets instead of serving stale copies of the code. Mix can handle this for you using the `version` method.

The `version` method will automatically append a unique hash to the filenames of all compiled files, allowing for more convenient cache busting:

    mix.js('resources/assets/js/app.js', 'public/js')
       .version();
    

After generating the versioned file, you won't know the exact file name. So, you should use Laravel's global `mix` function within your [views](/docs/{{version}}/views) to load the appropriately hashed asset. The `mix` function will automatically determine the current name of the hashed file:

    <link rel="stylesheet" href="{{ mix('/css/app.css') }}">
    

Because versioned files are usually unnecessary in development, you may instruct the versioning process to only run during `npm run production`:

    mix.js('resources/assets/js/app.js', 'public/js');
    
    if (mix.inProduction()) {
        mix.version();
    }
    

<a name="browsersync-reloading"></a>

## Browsersync Reloading

[BrowserSync](https://browsersync.io/) can automatically monitor your files for changes, and inject your changes into the browser without requiring a manual refresh. You may enable support by calling the `mix.browserSync()` method:

    mix.browserSync('my-domain.dev');
    
    // Or...
    
    // https://browsersync.io/docs/options
    mix.browserSync({
        proxy: 'my-domain.dev'
    });
    

You may pass either a string (proxy) or object (BrowserSync settings) to this method. Next, start Webpack's dev server using the `npm run watch` command. Now, when you modify a script or PHP file, watch as the browser instantly refreshes the page to reflect your changes.

<a name="environment-variables"></a>

## Environment Variables

You may inject environment variables into Mix by prefixing a key in your `.env` file with `MIX_`:

    MIX_SENTRY_DSN_PUBLIC=http://example.com
    

After the variable has been defined in your `.env` file, you may access via the `process.env` object. If the value changes while you are running a `watch` task, you will need to restart the task:

    process.env.MIX_SENTRY_DSN_PUBLIC
    

<a name="notifications"></a>

## Notifications

When available, Mix will automatically display OS notifications for each bundle. This will give you instant feedback, as to whether the compilation was successful or not. However, there may be instances when you'd prefer to disable these notifications. One such example might be triggering Mix on your production server. Notifications may be deactivated, via the `disableNotifications` method.

    mix.disableNotifications();