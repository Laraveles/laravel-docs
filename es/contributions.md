# Guía de contribución

- [Reporte de errores](#bug-reports)
- [Discusión del desarrollo del *core*](#core-development-discussion)
- [¿Qué rama? – *Branch*](#which-branch)
- [Vulnerabilidades de seguridad](#security-vulnerabilities)
- [Estilo de código](#coding-style) 
    - [PHPDoc](#phpdoc)
    - [StyleCI](#styleci)

<a name="bug-reports"></a>

## Reporte de errores

Para estimular la colaboración activa, Laravel recomienda *pull requests* encarecidamente, no simplemente informar de errores. Los "reportes de errores" se pueden enviar además en forma de *pull request* conteniendo un test fallido.

Sin embargo, si se presenta un reporte de error, el caso debe contener un título y una descripción clara del problema. También se debe incluir tanta información relevante como sea posible y ejemplos de código que demuestren el problema. El objetivo de un reporte es hacer fácil para ti - y otros - replicar el error y desarrollar una solución.

Recordar, los reportes se crean con la esperanza de que otros con el mismo problema sean capaces de colaborar contigo en su resolución. No se debe esperar que un reporte obtenga actividad automáticamente o que otros vengan a solucionarlo. Reportar un error sirve para ayudarnos a nosotros mismos y a otros a comenzar el camino para su solución.

El código fuente de Laravel se mantiene en GitHub, y hay repositorios para cada uno de los proyectos de Laravel:

<div class="content-list">
  <ul>
    <li>
      <a href="https://github.com/laravel/laravel">Laravel Application</a>
    </li>
    <li>
      <a href="https://github.com/laravel/art">Laravel Art</a>
    </li>
    <li>
      <a href="https://github.com/laravel/docs">Laravel Documentation</a>
    </li>
    <li>
      <a href="https://github.com/laravel/cashier">Laravel Cashier</a>
    </li>
    <li>
      <a href="https://github.com/laravel/cashier-braintree">Laravel Cashier para Braintree</a>
    </li>
    <li>
      <a href="https://github.com/laravel/envoy">Laravel Envoy</a>
    </li>
    <li>
      <a href="https://github.com/laravel/framework">Laravel Framework</a>
    </li>
    <li>
      <a href="https://github.com/laravel/homestead">Laravel Homestead</a>
    </li>
    <li>
      <a href="https://github.com/laravel/settler">Laravel Homestead Build Scripts</a>
    </li>
    <li>
      <a href="https://github.com/laravel/horizon">Laravel Horizon</a>
    </li>
    <li>
      <a href="https://github.com/laravel/passport">Laravel Passport</a>
    </li>
    <li>
      <a href="https://github.com/laravel/scout">Laravel Scout</a>
    </li>
    <li>
      <a href="https://github.com/laravel/socialite">Laravel Socialite</a>
    </li>
    <li>
      <a href="https://github.com/laravel/laravel.com">Laravel Website</a>
    </li>
  </ul>
</div>

<a name="core-development-discussion"></a>

## Discusión del desarrollo del *core*

Puede proponer nuevas características o mejoras al comportamiento de Laravel en el [tablero de issues](https://github.com/laravel/internals/issues) de *Laravel Internals*. Si propone una nueva característica, implemente, al menos, parte del código necesario para completarla.

El debate sobre errores, nuevas características e implementaciones de características existentes tienen lugar en el canal `#internals` del equipo de Slack [LaraChat](https://larachat.co). Taylor Otwell, el encargado de Laravel, está normalmente presente en el canal de lunes a viernes de 8:00-17:00 (UTC-06:00 o America/Chicago) y esporádicamente en otros momentos.

<a name="which-branch"></a>

## ¿Qué rama? – *Branch*

**Todas** las correcciones de errores deben enviarse al último *branch* (rama) estable o al *branch* LTS (5.5). Las correcciones **nunca** se deben enviar a la rama `master` a menos que se refieran a una característica que únicamente exista en la próxima versión.

Características **menores** que son **totalmente compatibles** desde versiones anteriores a la versión actual de Laravel se deben enviar a la rama de la última versión estable.

Características **mayores** se deben enviar siempre a la rama `master`, el cual contiene la próxima versión de Laravel.

Si se duda sobre si una característica se califica como menor o mayor, por favor preguntar a Taylor Otwell en el canal `#internals` de Slack [LaraChat](https://larachat.co).

<a name="security-vulnerabilities"></a>

## Vulnerabilidades de seguridad

Si se descubre una vulnerabilidad de seguridad en Laravel, por favor enviar un e-mail a Taylor Otwell a <taylor@laravel.com>. Se abordarán sin demora todas las vulnerabilidades de seguridad.

<a name="coding-style"></a>

## Estilo de codigo

Laravel sigue el estándar de código [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) y el estándar de carga automática [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md).

<a name="phpdoc"></a>

### PHPDoc

A continuación se muestra un ejemplo válido de un bloque de documentación de Laravel. Se debe tener en cuenta que al atributo `@param` le siguen dos espacios, el tipo de argumento, dos espacios más y finaliza con el nombre de la variable:

    /**
     * Register a binding with the container.
     *
     * @param  string|array  $abstract
     * @param  \Closure|string|null  $concrete
     * @param  bool  $shared
     * @return void
     */
    public function bind($abstract, $concrete = null, $shared = false)
    {
        //
    }
    

<a name="styleci"></a>

### StyleCI

¡No se preocupe si el estilo de su código no es perfecto! [StyleCI](https://styleci.io/) automáticamente fusionará cualquier corrección de estilo en el repositorio de Laravel después de fusionar las *pull requests*. Esto nos permite centrarnos en el contenido de la contribución y no en el estilo del código.