# *Scaffolding* JavaScript & CSS

- [Introducción](#introduction)
- [Escribir CSS](#writing-css)
- [Escribir JavaScript](#writing-javascript) 
    - [Escribiendo componentes Vue](#writing-vue-components)
    - [Usando React](#using-react)

<a name="introduction"></a>

## Introducción

Si bien Laravel no dicta qué preprocesadores JavaScript o CSS utilizar, sí proporciona un punto de partida básico utilizando [Bootstrap](https://getbootstrap.com/) y [Vue](https://vuejs.org) que será útil para muchas aplicaciones. Por defecto, Laravel usa [NPM](https://www.npmjs.org) para instalar ambos paquetes frontend.

#### CSS

[Laravel Mix](/docs/{{version}}/mix) proporciona una API limpia y expresiva sobre la compilación de SASS o Less, que son extensiones de CSS simple que añaden variables, mezclas y otras características poderosas que hacen que trabajar con CSS sea mucho más agradable. En este documento, discutiremos brevemente la compilación CSS en general; sin embargo, debe consultar la documentación completa de [Laravel Mix](/docs/{{version}}/mix) para obtener más información sobre la compilación de SASS o Less.

#### JavaScript

Laravel no requiere que utilice un framework o biblioteca de JavaScript específico para construir sus aplicaciones. De hecho, no tiene que usar JavaScript. Sin embargo, Laravel incluye algunos scaffolding básicos para facilitar el inicio de la escritura de JavaScript moderno usando la biblioteca [Vue](https://vuejs.org). Vue proporciona una API expresiva para construir aplicaciones JavaScript robustas usando componentes. Al igual que con CSS, podemos usar Laravel Mix para compilar fácilmente componentes JavaScript en un solo archivo listo para el navegador.

#### Eliminar el *scaffolding* de frontend

Si desea retirar el *scaffolding* frontend de su aplicación, puede utilizar el comando Artisan `preset`. Este comando, cuando se combina con la opción `none`, eliminará el *scaffolding* Bootstrap y Vue de su aplicación, dejando sólo un archivo SASS en blanco y algunas librerías comunes de utilidades JavaScript:

    php artisan preset none
    

<a name="writing-css"></a>

## Escribiendo CSS

El archivo `package.json` de Laravel incluye el paquete `bootstrap-sass` para ayudarle a empezar a crear prototipos del frontend de su aplicación usando Bootstrap. Sin embargo, no dude en añadir o eliminar paquetes del archivo `package.json` según sea necesario para su propia aplicación. Usted no está obligado a utilizar el framework Bootstrap para construir su aplicación Laravel - simplemente se proporciona como un buen punto de partida para aquellos que elijan utilizarlo.

Antes de compilar su CSS, instale las dependencias frontend de su proyecto utilizando [Node package manager (NPM)](https://www.npmjs.org):

    npm install
    

Una vez instaladas las dependencias usando `npm install`, puede compilar sus archivos SASS a CSS plano usando [Laravel Mix](/docs/{{version}}/mix#working-with-stylesheets). El comando `npm run dev` procesará las instrucciones en su archivo `webpack.mix.js`. Normalmente, su CSS compilado se ubicará en el directorio `public/css`:

    npm run dev
    

Por defecto el archivo `webpack.mix.js` incluido con Laravel compilará el archivo SASS `resources/assets/sass/app.scss`. Este archivo `app.scss` importa un archivo de variables SASS y carga Bootstrap, que proporciona un buen punto de partida para la mayoría de las aplicaciones. Siéntase libre de personalizar el archivo `app.scss` si usted lo desea o incluso utilizar un pre-procesador completamente diferente: [configuring Laravel Mix](/docs/{{version}}/mix).

<a name="writing-javascript"></a>

## Escribiendo JavaScript

Todas las dependencias JavaScript requeridas por su aplicación se pueden encontrar en el archivo `package.json` en el directorio raíz del proyecto. Este archivo es similar al archivo `composer.json` excepto que especifica dependencias JavaScript en lugar de dependencias PHP. Puede instalar estas dependencias utilizando [Node package manager (NPM)](https://www.npmjs.org):

    npm install
    

> {tip} Por defecto, el archivo `package.json` incluye algunos paquetes como `vue` y `axios` para ayudarle a empezar a crear su aplicación JavaScript. Siéntase libre de agregar o quitar paquetes necesarios para su aplicación del archivo `package.json`.

Una vez instalados los paquetes, puede usar el comando `npm run dev` para [compilar sus recursos](/docs/{{version}}/mix). Webpack es un empaquetador de módulos para aplicaciones JavaScript modernas. Cuando ejecuta el comando `npm run dev`, Webpack ejecutará las instrucciones en su archivo `webpack.mix.js`:

    npm run dev
    

De forma predeterminada, el archivo Laravel `webpack.mix.js` compila su SASS y el archivo `resources/assets/js/app.js`. Dentro del archivo `app.js` puede registrar sus componentes Vue o, si prefiere un framework diferente, configurar su propia aplicación JavaScript. Su JavaScript compilado se ubicará normalmente en el directorio `public/js`.

> {tip} El archivo `app.js` cargará el archivo `resources/assets/js/bootstrap.js` que inicia y configura Vue, Axios, jQuery y todas las demás dependencias JavaScript. Si tiene dependencias JavaScript adicionales para configurar, puede hacerlo en este archivo.

<a name="writing-vue-components"></a>

### Escribiendo componentes Vue

Por defecto, las aplicaciones de Laravel nuevas contienen un componente Vue `ExampleComponent.vue` situado en el directorio `resources/assets/js/components`. El archivo `ExampleComponent.vue` es un ejemplo de [componente vue de archivo simple](https://vuejs.org/guide/single-file-components) que define su plantilla JavaScript y HTML en el mismo archivo. Los componentes de un solo archivo proporcionan un enfoque muy conveniente para la creación de aplicaciones basadas en JavaScript. El componente de ejemplo está registrado en su archivo `app.js`:

    Vue.component(
        'example-component',
        require('./components/ExampleComponent.vue')
    );
    

Para utilizar el componente de su aplicación, simplemente puede colocarlo en una de sus plantillas HTML. Por ejemplo, después de ejecutar el comando Artisan `make:auth` para crear las vistas de autenticación y registro de su aplicación, podría colocar el componente en la plantilla Blade `home.blade.php`:

    @extends('layouts.app')
    
    @section('content')
        <example-component></example-component>
    @endsection
    

> {tip} Recuerde, debería ejecutar el comando `npm run dev` cada vez que cambie un componente Vue. O bien, puede ejecutar el comando `npm run watch` para monitorear y recompilar automáticamente sus componentes cada vez que se modifiquen.

Por supuesto, si está interesado en aprender más sobre la escritura de componentes Vue, debe leer la [Documentación Vue](https://vuejs.org/guide/), que proporciona una visión general completa y fácil de leer de todo el framework Vue.

<a name="using-react"></a>

### Usar React

Si usted prefiere usar React para construir su aplicación JavaScript, Laravel hace que sea fácil intercambiar el *scaffolding* Vue con el *scaffolding* React. En cualquier aplicación de Laravel nueva, puede usar el comando `preset` con la opción `react`:

    php artisan preset react
    

Este comando único eliminará el *scaffolding* Vue y lo reemplazará con el *scaffolding* React, incluyendo un componente de ejemplo.