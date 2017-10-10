# Ciclo de Vida de la Petición

- [Introduccion](#introduction)
- [Visión General del Ciclo de Vida](#lifecycle-overview)
- [Objetivo Service Providers](#focus-on-service-providers)

<a name="introduction"></a>

## Introduccion

Cuando se usa cualquier herramienta en el "mundo real", te sientes mas seguro si entiendes cómo funciona. El desarrollo de aplicaciones no es diferente. Cuando entiendes como funcionan las herramientas de desarrollo, te sientes mas cómodo y seguro usándolas.

El objetivo de este documento es dar una buena visión general, y de alto nivel, de cómo funciona el framework Laravel. Al conocer mejor el framework en general, todo parece menos "mágico" y estará más seguro construyendo aplicaciones. Si no entiendes a la primera todos los términos, ¡no te desanimes! Sólo trata de obtener una idea básica de que está sucediendo, y el conocimiento crecerá a medida que se vayan explorando otras secciones de la documentación.

<a name="lifecycle-overview"></a>

## Visión General del Ciclo de Vida

### Lo Primero

El punto de entrada de todas las peticiones a una aplicación Laravel es el archivo `public/index.php`. Todas las peticiones son dirigidas a este fichero por la configuración del servidor web (Apache / Nginx). El archivo `index.php` no contiene mucho código. Simplemente es el inicio de la carga del resto del framework.

El fichero `index.php` lee las definiciones de autoloader generadas por Composer, y entonces recupera una instancia de la aplicación Laravel desde el script `bootstrap/app.php`. La primera acción llevada a cabo por el propio Laravel es crear una instancia de la aplicación / [service container](/docs/{{version}}/container).

### Núcleos HTTP / Console

Seguidamente, la petición entrante es enviada al núcleo HTTP o al núcleo de consola, dependiendo el tipo de petición que esté entrando a la aplicación. Estos dos núcleos sirven como ubicación central de todas las peticiones que pasen a través. Por ahora, centrémonos en el núcleo HTTP, localizado en el fichero `app/Http/Kernel.php`.

El núcleo HTTP hereda de la clase `Illuminate\Foundation\Http\Kernel`, la cual define un array de `bootstrappers` que se procesarán antes de que la petición sea ejecutada. La misión de estos "bootstrapers" es configurar el manejo de errores, el logging, [detectar el entorno de la aplicación](/docs/{{version}}/configuration#environment-configuration) y realizar otras tareas que son necesarias antes de que se procese la petición actual.

El núcleo HTTP también define una lista de HTTP [middleware](/docs/{{version}}/middleware) que todas las peticiones deben pasar antes de que sean procesadas por la aplicación. Estos "middleware" manejan la lectura/escritura de la [sesión HTTP](/docs/{{version}}/session), detectan si la aplicación está en modo mantenimiento, [verifica el token CSRF](/docs/{{version}}/csrf), y más.

La firma del método `handle` del núcleo HTTP es bastante simple: recibe `Request` y devuelve `Response`. Pensar en el Núcleo como una gran caja negra que representa la totalidad de la aplicación. Aliméntalo con "HTTP requests" y devolverá "HTTP responses".

#### Proveedores de Servicio (Service Providers)

Una de las acciones más importantes de la inicialización del núcleo es la carga de los [service providers](/docs/{{version}}/providers) de la aplicación. Todos los "service providers" de la aplicación están configurados dentro del archivo de configuración `config/app.php` en el array `providers`. Primero, para todos los providers se llamará al método `register`, una vez todos los providers hayan sido registrados se llamará al método `boot`.

Los "service providers" son responsables de la inicialización de los diversos componentes del framework, como la base de datos, colas, validaciones y los componentes de routing. Puesto que inicializan y configuran cada característica ofrecida por el framework, los "service providers" son el aspecto más importante del proceso de inicialización de Laravel.

#### Lanzar la Petición

Una vez la aplicación se ha inicializado y todos los "service providers" han sido registrados la `Petición` (Request) será traspasada al router para su lanzamiento. El router lanzará la request a una ruta o a un controlador, así como ejecutará cualquier middleware asociado a la ruta.

<a name="focus-on-service-providers"></a>

## Objetivo Service Providers

Los Service Providers son verdaderamente la clave para inicializar una aplicación Laravel. La instancia de la aplicación está creada, los service providers están registrados y la petición es enviada a la aplicación inicializada. ¡Es así de simple!

Tener una firme convicción de como una aplicación Laravel se construye e inicializa a través de los service providers es muy valioso. Por supuesto, los service providers por defecto de la aplicación están almacenados en la carpeta `app/Providers`.

Por defecto, el `AppServiceProvider` tiene poco contenido. Este provider es un buen lugar donde añadir tus propias inicializaciones y los enlaces a service containers. Por supuesto que para grandes aplicaciones puede querer crear varios service providers, cada uno con una pequeña parte de la inicialización.