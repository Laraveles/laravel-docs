# Despliegue (Deployment)

- [Introducción](#introduction)
- [Configuración del Servidor](#server-configuration) 
    - [Nginx](#nginx)
- [Optimización](#optimization) 
    - [Optimizacion de Autoloader](#autoloader-optimization)
    - [Optimización de la Configuración](#optimizing-configuration-loading)
    - [Optimización de las Rutas](#optimizing-route-loading)
- [Desplegar con Forge](#deploying-with-forge)

<a name="introduction"></a>

## Introducción

Cuando se tiene una aplicación de Laravel lista para el despliegue en producción, hay algunas cosas importantes que se pueden hacer para asegurarse de que la misma se ejecute tan eficientemente como sea posible. En este documento, se cubrirán algunos puntos de partida para asegurar que la aplicación se despliegue correctamente.

<a name="server-configuration"></a>

## Configuración del Servidor

<a name="nginx"></a>

### Nginx

Si se despliega la aplicación dentro de un servidor que esta corriendo Nginx, se puede usar el siguiente archivo de configuración como punto de partida para el servidor web. Lo más probable es que este archivo necesite ser personalizado dependiendo de la configuración del servidor. Si se necesita asistencia para el manejo de servidores, se puede considerar usar un servicio como [Laravel Forge](https://forge.laravel.com):

    server {
        listen 80;
        server_name example.com;
        root /example.com/public;
    
        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Content-Type-Options "nosniff";
    
        index index.html index.htm index.php;
    
        charset utf-8;
    
        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }
    
        location = /favicon.ico { access_log off; log_not_found off; }
        location = /robots.txt  { access_log off; log_not_found off; }
    
        error_page 404 /index.php;
    
        location ~ \.php$ {
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass unix:/var/run/php/php7.1-fpm.sock;
            fastcgi_index index.php;
            include fastcgi_params;
        }
    
        location ~ /\.(?!well-known).* {
            deny all;
        }
    }
    

<a name="optimization"></a>

## Optimización

<a name="autoloader-optimization"></a>

### Optimizacion de Autoloader

Cuando se despliega a producción, hay que asegurarse de optimizar el *autoloader map* de Composer para que pueda encontrar rápidamente el archivo adecuado para cargar para una clase dada:

    composer install --optimize-autoloader
    

> {tip} Ademas de optimizar el *autoloader*, siempre se debe incluir el archivo `composer.lock` dentro del repositorio del proyecto. Las dependencias del proyecto se pueden instalar mucho mas rápido cuando el archivo `composer.lock` esta presente.

<a name="optimizing-configuration-loading"></a>

### Optimización de la configuración

Cuando se despliega una aplicación en producción, hay que recordar ejecutar el comando de Artisan `config:cache` durante el proceso de despliegue:

    php artisan config:cache
    

Este comando permite combinar todos los archivos de configuración de Laravel en uno solo archivo cacheado, lo que reduce en gran medida el número de peticiones que el framework debe realizar al sistema de archivos al cargar los valores de configuración.

<a name="optimizing-route-loading"></a>

### Optimización de las Rutas

Cuando se despliega una aplicación en producción, hay que recordar ejecutar el comando de artisan `route:cache` durante el proceso de despliegue:

    php artisan route:cache
    

Este comando reduce todos los registros de las rutas en una sola llamada dentro de un archivo en caché, mejorando el rendimiento del registro de rutas al momento de registrar cientos de las mismas.

> {note} Esta característica usa la serialización de PHP, y sólo puede almacenar en caché las rutas para aplicaciones que utilizan exclusivamente rutas basadas en controladores. PHP no puede serializar Clousures.

<a name="deploying-with-forge"></a>

## Desplegar con Forge

Si no se desea gestionar la configuración de un servidor o no se siente cómodo configurando todos los servicios necesarios para ejecutar una aplicación robusta de Laravel, se puede utilizar como una buena alternativa el servicio de [Laravel Forge ](https://forge.laravel.com).

Laravel Forge puede crear servidores en varios proveedores de infraestructura como lo son DigitalOcean, Linode, AWS y otros. Ademas, Forge instala y gestiona todas las herramientas necesarias para construir una aplicación de Laravel robusta, tal como Nginx, MySQL, Redis, Memcached, Beanstalk, y más.