# Laravel Homestead

- [Introducción](#introduction)
- [Instalación & configuración](#installation-and-setup) 
    - [Primeros pasos](#first-steps)
    - [Configurar Homestead](#configuring-homestead)
    - [Levantar la box de Vagrant](#launching-the-vagrant-box)
    - [Instalación por proyecto](#per-project-installation)
    - [Instalando MariaDB](#installing-mariadb)
    - [Instalando *ElasticSearch*](#installing-elasticsearch)
    - [Alias](#aliases)
- [Uso diario](#daily-usage) 
    - [Accediendo globalmente a Homestead](#accessing-homestead-globally)
    - [Conectándo a través de SSH](#connecting-via-ssh)
    - [Conectar a bases de datos](#connecting-to-databases)
    - [Añadir nuevos sitios](#adding-additional-sites)
    - [Variables de entorno](#environment-variables)
    - [Configurar programaciones *cron*](#configuring-cron-schedules)
    - [Configurar Mailhog](#configuring-mailhog)
    - [Puertos](#ports)
    - [Compartir el entorno](#sharing-your-environment)
    - [Múltiples versiones de PHP](#multiple-php-versions)
- [Intefaces de red](#network-interfaces)
- [Actualizar Homestead](#updating-homestead)
- [Versiones antiguas](#old-versions)
- [Configuración específica del proveedor](#provider-specific-settings) 
    - [VirtualBox](#provider-specific-virtualbox)

<a name="introduction"></a>

## Introducción

Laravel se esfuerza por hacer toda la experiencia de desarrollo de PHP agradable, incluyendo el entorno de desarrollo local. [Vagrant](https://www.vagrantup.com) provee una simple y elegante manera de gestionar y suministrar máquinas virtuales.

Laravel Homestead es una *Vagrant box* oficial que le provee de un maravilloso entorno de desarrollo sin requerirle instalar PHP, un servidor web, y cualquier otras aplicaciones de servidor en su máquina local. No más preocupaciones acerca de estropear su sistema operativo! Las "boxes" de Vagrant son completamente desechables. Si algo sale mal, ¡se puede destruir la box y crearla nuevamente en cuestión de minutos!

Homestead se puede ejecutar sobre cualquier sistema Windows, Mac o Linux e incluye el servidor web Nginx, PHP 7.1, PHP 7.0, PHP 5.6, MySQL, PostgreSQL, Redis, Memcached, Node, y todas las características necesarias para desarrollar aplicaciones con Laravel.

> {note} Si está utilizando Windows, podría necesitar activar la virtualización por *hardware* (VT-x). Normalmente, ésta puede activarse a través de su BIOS. Si está utilizando Hyper-V en una sistema UEFI, además puede necesitar desactivar Hyper-V para acceder a VT-x.

<a name="included-software"></a>

### Software Incluído

<div class="content-list">
  <ul>
    <li>
      Ubuntu 16.04
    </li>
    <li>
      Git
    </li>
    <li>
      PHP 7.1
    </li>
    <li>
      PHP 7.0
    </li>
    <li>
      PHP 5.6
    </li>
    <li>
      Nginx
    </li>
    <li>
      MySQL
    </li>
    <li>
      MariaDB
    </li>
    <li>
      Sqlite3
    </li>
    <li>
      PostgreSQL
    </li>
    <li>
      Composer
    </li>
    <li>
      Node (With Yarn, Bower, Grunt, and Gulp)
    </li>
    <li>
      Redis
    </li>
    <li>
      Memcached
    </li>
    <li>
      Beanstalkd
    </li>
    <li>
      Mailhog
    </li>
    <li>
      ngrok
    </li>
  </ul>
</div>

<a name="installation-and-setup"></a>

## Instalación & Configuración

<a name="first-steps"></a>

### Primeros Pasos

Antes de arrancar tu entorno de *Homestead*, deberá instalar [VirtualBox 5.1](https://www.virtualbox.org/wiki/Downloads) [VMWare](https://www.vmware.com), or [Parallels](https://www.parallels.com/products/desktop/) así como [Vagrant](https://www.vagrantup.com/downloads.html). Todos estos programas poseen instaladores visuales de fácil uso para los todos los sistemas operativos más populares.

Par usar como proveedor VMWare, necesitará adquirir VMWare Fusion o Workstation y el complemento [VMware Vagrant](https://www.vagrantup.com/vmware). Aunque no es gratuito, VMWare puede proporcionar un mejor rendimiento en carpetas compartidas, sin configuración extra.

Para utilizar el proveedor Parallels, necesitará instalar el complento [Parallels Vagrant](https://github.com/Parallels/vagrant-parallels). Este es gratis.

#### Instalando Homestead con Vagrant

Una vez que VirtualBox / VMware y Vagrant han sido instalados, debe agregar la *box* de `laravel/homestead` a la instalación de Vagrant, utilizando el siguiente comando desde su consola. Tomará algunos minutos para que la box se descargue, esto dependerá de la velocidad de la conexión a internet:

    vagrant box add laravel/homestead
    

Si este comando falla, asegúrese de que su instalación de Vagrant está actualizada.

#### Instalando Homestead

Puede instalar Homestead simplemente clonando el repositorio. Considere clonar el repositorio dentro de una carpeta llamada `Homestead` en su directorio *home*, así la Homestead *box* servirá como host para todos sus projectos Laravel:

    cd ~
    
    git clone https://github.com/laravel/homestead.git Homestead
    

Debe consultar una versión etiquetada de Homestead ya que la rama `master</ 0> puede no ser siempre estable. Puede encontrar la última versión estable en la página <a href="https://github.com/laravel/homestead/releases">GitHub Release</a>:</p>

<pre><code>cd Homestead

// Clone the desired release...
git checkout v6.5.0
`</pre> 

Una vez haya clonado el repositorio Homestead, ejecute el comando `bash init.sh` desde el directorio Homestead para crear el archivo de configuración `Homestead.yaml`. El fichero `Homestead.yaml` será colocado en el directorio Homestead:

    // Mac / Linux...
    bash init.sh
    
    // Windows...
    init.bat
    

<a name="configuring-homestead"></a>

### Configurar Homestead

#### Establecer proveedor

La clave `proveedor` en su fichero `Homestead.yaml` indica que proveedor Vagrant debería ser usado: `virtualbox`, `vmware_fusion`, `vmware_workstation`, o `parallels`. Puede configurar el proveedor que prefiera:

    provider: virtualbox
    

#### Configurar directorios compartidos

La propiedad `folders` del archivo `Homestead.yaml` enumera todas las carpetas que dese compartir con el entorno Homestead. Cuando los ficheros en esas carpetas cambien, se mantendrán sincronizados entre su máquina local y el entorno Homestead. Puede configurar tantas carpetas como sean necesarias:

    folders:
    
        - map: ~/code
          to: /home/vagrant/code
    

Si sólo está creando algunos sitios, este mapeo genérico funcionará bien. No obstante, cuando el número de sitios continúe creciendo, puede empezar a experimentar problemas de velocidad. Este problema puede ser muy exasperante en máquinas de bajo rendimiento o projectos que contengan una gran cantidad de ficheros. Si experimenta este problema, pruebe a mapear cada proyecto sobre su propia carpeta Vagrant:

    folders:
    
        - map: ~/code/project1
          to: /home/vagrant/code/project1
    
        - map: ~/code/project2
          to: /home/vagrant/code/project2
    

Para activar [NFS](https://www.vagrantup.com/docs/synced-folders/nfs.html), sólo necesita añadir un simple indicador a su configuración de carpetas sincronizadas:

    folders:
    
        - map: ~/code
          to: /home/vagrant/code
          type: "nfs"
    

> {note} Cuando utilice NFS, debería considerar instalar el complemento [vagrant-bindfs](https://github.com/gael-ian/vagrant-bindfs). Este complemento mantendrá los permisos de usuario y grupo correctos para los ficheros y directorios dentro de *Homestead box*.

Puede pasar cualquiera de las opciones soportadas por las [carpetas sincronizadas](https://www.vagrantup.com/docs/synced-folders/basic_usage.html) de Vagrant enumerándolas en la sección de `opciones`:

    folders:
    
        - map: ~/code
          to: /home/vagrant/code
          type: "rsync"
          options:
              rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
              rsync__exclude: ["node_modules"]
    

#### Configurar sitios Nginx

¿No está familiarizado con Nginx? No hay problema. La propiedad `sites` le permite de una manera sencilla mapear un "dominio" a una carpeta en su entorno Homestead. Un ejemplo de configuración de una sitio web está incluído en el archivo `Homestead.yaml`. De nuevo, puede añadir tantos sitios a su entorno Homestead como sean necesarios. Homestead puede servir como un entorno virtualizado para cada proyecto de Laravel en el que esté trabajando:

    sites:
    
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
    

Si cambia la propiedad `sites` después de provisionar *Homestead box*, debería volver a ejecutar `vagrant reload --provision` para actualizar la configuración de Nginx en la máquina virtual.

#### El archivo Hosts

Debe agregar los "dominios" para sus sitios web en Nginx en el fichero `hosts` en su máquina. El fichero `hosts` redireccionará las peticiones para sus sitios Homestead hacia su máquina Homestead. En Mac y Linux, este archivo está ubicado en `/etc/hosts`. En Windows el archivo se encuentra ubicado en `C:\Windows\System32\drivers\etc\hosts`. Las líneas que se deben añadir al archivo deben parecerse a las siguientes:

    192.168.10.10  homestead.test
    

Asegúrese que la dirección IP listada es la misma que está establecida en su fichero `Homestad.yaml`. Una vez haya agregado el domino a su fichero `hosts` y lanzado *Vagrant box* será capaz de acceder al sitio web a través de su navegador web:

    http://homestead.test
    

<a name="launching-the-vagrant-box"></a>

### Levantando *Vagrant box*

Una vez que ha editado `Homestead.yaml` a su gusto, ejecute el comando `vagrant up` desde el directorio de Homestead. Vagrant iniciará la máquina virtual y automáticamente configurará los directorios compartidos y sitios web en Nginx.

Para apagar la máquina, puede utilizar el comando `vagrant destroy --force`.

<a name="per-project-installation"></a>

### Instalación por proyecto

En lugar de instalar Homestead de manera global y compartir el mismo *Homestead box* entre todos sus proyectos, puede configurar una instancia de Homestead para cada proyecto que gestione. Instalando Homested para cada proyecto podrá beneficiarse si desea entregar el fichero `Vagrantfile` con su proyecto, permitiendo a otros trabajar en el proyecto usando simplemente `vagrant up`.

Para instalar Homestead directamente en su proyecto, se requiere la utilización de Composer:

    composer require laravel/homestead --dev
    

Una vez que Homestead ha sido instalado, utilizar el comando `make` para generar el fichero `VagrantFile` y `Homestead.yaml` en la raíz del proyecto. El comando `make` configurará automáticamente las directivas de los `sitios` y `carpetas` en el archivo `Homestead.yaml`.

Mac / Linux:

    php vendor/bin/homestead make
    

Windows:

    vendor\\bin\\homestead make
    

A continuación, ejecute el comando `vagrant up` en su terminal y desde su navegador, acceda a su proyecto en `http://homestead.test`. Recordar, todavía es necesario añadir una fila al archivo `/etc/hosts` para el dominio `homestead.test` o el dominio elegido.

<a name="installing-mariadb"></a>

### Instalando MariaDB

Si prefiere utilizar MariaDB en lugar de MySQL, puede agregar la opción `mariadb` a su fichero `Homestead.yaml`. Esta opción eliminará MySQL e instalará MariaDB. MariaDB sirve como reemplazo de MySQL así que deberías seguir utilizando el manejador de base dataos `mysql` en la configuración de la base de datos de su aplicación:

    box: laravel/homestead
    ip: "192.168.10.10"
    memory: 2048
    cpus: 4
    provider: virtualbox
    mariadb: true
    

<a name="installing-elasticsearch"></a>

### Instalación de *Elasticsearch*

Para instalar *ElasticSearch*, agrege la opción `elasticsearch` a su fichero `Homestead.yaml`. La instalación por defecto creará un cluster llamado 'homestead' y reservará para él 2Gb de memoria. No debería proporcionarle a *Elasticsearch* más de la mitad de la memoria destinada a su sistema operativo, asegúrese que su máquina *Homestead* tiene al menos 4Gb de memoria:

    box: laravel/homestead
    ip: "192.168.10.10"
    memory: 4096
    cpus: 4
    provider: virtualbox
    elasticsearch: true
    

<a name="aliases"></a>

### Alias

Podrá agregar alias *Bash* a su máquina *Homestead* modificando el fichero `aliases` dentro de su directorio Homestead:

    alias c='clear'
    alias ..='cd ..'
    

Después de haber actualizado el fichero `aliases`, debería recargar la máquina *Homestead* usando el comando `vagrant reload --provision`. Esto asegurará que sus nuevos alias están disponibles en la máquina.

<a name="daily-usage"></a>

## Uso diario

<a name="accessing-homestead-globally"></a>

### Accediendo globalmente a *Homestead*

A veces puede necesitar arrancar su máquina *Homestead* `vagrant up` desde cualquier parte de su sistema. Para conseguir esto en sistemas Mac / Linux agregando una función *Bash* a su perfil *Bash*. En Windows, puede realizar esto agregando un fichero "batch" a su `PATH`. Estos *scripts* le permitirán ejecutar cualquier comando Vagrant desde cualquier sitio en su sistema y automaticamente apuntará a su instalación *Homestead*:

#### Mac / Linux

    function homestead() {
        ( cd ~/Homestead && vagrant $* )
    }
    

Asegúrese de cambiar el camino `~Homestead` en la función por la situación de su instalación actual de *Homestead*. Una vez la función esté instalada, podrá ejecutar comandos como `homestead up` o `homestead ssh` desde cualquier parte de su sistema.

#### Windows

Cree un fichero batch `homestead.bat`, en cualquier parte de su máquina, con el siguiente contenido:

    @echo off
    
    set cwd=%cd%
    set homesteadVagrant=C:\Homestead
    
    cd /d %homesteadVagrant% && vagrant %*
    cd /d %cwd%
    
    set cwd=
    set homesteadVagrant=
    

Asegúrese de cambiar el camino de ejemplo `C:\Homestead` en el *script* a la situación actual de su instalación *Homestead*. Después de crear el fichero, agrege la situación a su `PATH`. Puede ahora ejecutar comandos como `homestead up` o `homestead ssh` desde cualquier parte de su sistema.

<a name="connecting-via-ssh"></a>

### Conectándo a través de SSH

Puede, usando SSH, entrar a su máquina virtual introduciendo en su terminal el comando `vagrant ssh` desde el directorio *Homestead*.

Pero probablemente necesitará frecuentemente entrar a su máquina utilizando SSH, considere agregar la "función" descrita a con anterioridad a su máquina anfritrión para usarla rápidamente.

<a name="connecting-to-databases"></a>

### Conectando a las bases de datos

La base de datos `homestead` está configurada para los sistemas MySQL y PostgreSQL desde el inicio. Para su conveniencia, el fichero `.env` configura el *framework* para utilizar esta base de datos desde el inicio.

Para conectar con su base de datos MySQL o PostgreSQL desde su cliente de base de datos en su sistema anfitrión, debería conectar a `127.0.0.1` y el puerto `33060` (MySQL) o `54320` (PostgreSQL). El nombre de usuario y contraseña para ambas bases de datos es `homestead` / `secret`.

> {note} Debería solo usar estos puertos no estándar cuando conecte con bases de datos desde su máquina anfitrión. Utilizará los puertos por defecto 3306 y 5432 en su configuración Laravel de la base de datos dado que Laravel está ejecutándose *dentro* de una máquina virtual.

<a name="adding-additional-sites"></a>

### Añadiendo *Sitios* adicionales

Una vez su entorno *Homestead* está aprovisionado y corriendo, puede querer agregar *sitios Nginx* adicionales para sus aplicaciones Laravel. Puede ejecutar cuantas instalaciones Laravel desee en un entorno único de *Homestead*. Para agregar *sitios* adicionales, simplemente agréguelos a su fichero `Homestead`:

    sites:
    
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
        - map: another.test
          to: /home/vagrant/code/another/public
    

Si *Vagrant* no está gestionando su fichero "hosts" automáticamente, además deberá añadir el nuevo *sitio* a ese fichero:

    192.168.10.10  homestead.test
    192.168.10.10  another.test
    

Una vez que el *sitio* ha sido agregado, ejecute el comando `vagrant reload --provision` desde su directorio *Homestead</0>.</p> 

<a name="site-types"></a>

#### Tipos de *sitios*

*Homestead* soporta varios tipos de *sitios* los cuales le permiten fácilmente ejecutar projectos que no estén basados en Laravel. Por ejemplo, podemos, fácilmente, agregar una aplicación *Symfony* a *Homestead* usando el tipo de *sitio* `symfony2`:

    sites:
    
        - map: symfony2.test
          to: /home/vagrant/code/Symfony/web
          type: symfony2
    

Los tipos de *sitios* disponibles son: `apache`, `laravel` (el defecto), `proxy`, `silverstripe`, `statamic`, `symfony2`, y `symfony4`.

<a name="site-parameters"></a>

#### Parámetros del *sitio*

Puede agregar a *Nginx* valores `fastcgi_param` adicionales a su *sitio* a través de la directiva de *sitio* `params`. Por ejemplo, nosotros agregamos un parámetro `FOO` con el valor de `BAR`:

    sites:
    
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
          params:
              - key: FOO
                value: BAR
    

<a name="environment-variables"></a>

### Variables de entorno

Se pueden establecer variables de entorno globales añadiéndolas al archivo `Homestead.yaml`:

    variables:
    
        - key: APP_ENV
          value: local
        - key: FOO
          value: bar
    

Tras actualizar `Homestead.yaml`, asegúrese de re-provisionar la máquina ejecutando `vagrant reload --provision`. Esto actualizará la configuración de PHP-FPM para todas las versiones de PHP instaladas y actualizará el entorno para el usuario de `vagrant`.

<a name="configuring-cron-schedules"></a>

### Configurando programaciones *Cron*

Laravel ofrece una manera sencilla de [agendar tareas al Cron](/docs/{{version}}/scheduling), agregando el comando de Artisan `schedule:run` especificando que se ejecute cada minuto. El comando `schedule:run` examinará la programación de trabajo definida en su clase `App\Console\Kernel` para determinar cuales trabajos deberían ser ejecutados.

Si quieres que el comando `schedule:run` corra para un sitio de Homestead en particular, deberás asignar la opción `schedule` a `true` cuando definas el sitio:

    sites:
    
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
          schedule: true
    

La tarea Cron para el sitio será definida en directorio `/etc/cron.d` de tu máquina virtual.

<a name="configuring-mailhog"></a>

### Configurando Mailhog

*Mailhog* le permite fácilmente capturar sus correos electrónicos salientes y examinarlos sin realmente enviarlos a sus destinatarios. Para empezar, actualice su fichero `.env` para usar la siguiente configuración de correo:

    MAIL_DRIVER=smtp
    MAIL_HOST=localhost
    MAIL_PORT=1025
    MAIL_USERNAME=null
    MAIL_PASSWORD=null
    MAIL_ENCRYPTION=null
    

<a name="ports"></a>

### Puertos

Por defecto, los siguientes puertos se redirigen al entorno Homestead:

- **SSH:** 2222 &rarr; redirigido a 22
- **ngrok UI:** 4040 &rarr; redirigido a 4040
- **HTTP:** 8000 &rarr; redirigido a 80
- **HTTPS:** 44300 &rarr; redirigido a 443
- **MySQL:** 33060 &rarr; redirigido a 3306
- **PostgreSQL:** 54320 &rarr; redirigido a 5432
- **Mailhog:** 8025 &rarr; redirigido a 8025

#### Redirigir Otros Puertos

Si desea, se pueden redirigir puertos adicionales a la box de Vagrant, así como especificar su protocolo:

    ports:
    
        - send: 50000
          to: 5000
        - send: 7777
          to: 777
          protocol: udp
    

<a name="sharing-your-environment"></a>

### Compartiendo tu entorno

A veces puede desear compartir lo que está trabajando con otros compañeros o con un cliente. Vagrant integra un sistema que a través de `vagrant share` soporta esto, no obstante, esto no funcionará si tiene multiples *sitios* configurados en su fichero `Homestead.yaml`.

Para resolver este problema, *Homestead* incluye su propio comando `share`. Para empezar, conecte a través de SSH con su máquina *Homestead* con el comando `vagrant ssh` y ejecute `share homestead.test`. Esto compartirá el *sitio* `homestead.test` desde su fichero de configuración `Homestead.yaml`. Por supuesto, puede sustituir `homestead.test` por cualquiera de los otros sitios configurados:

    share homestead.test
    

Después de ejecutar el comando, verá aparecer una pantalla *Ngrok* conteniendo el registro de actividad y las URLs públicamente accesibles para el *sitio* compartido. Si quisiera especificar una region especifica, subdominio o cualquier otra opción *Ngrok* en tiempo de ejecución, puede agregarlas a su comando `share`:

    share homestead.test -region=eu -subdomain=laravel
    

> {note} Recuerde que *Vagrant* es inherentemente inseguro y está exponiendo su máquina virtual a Internet cuando ejecuta el comando `share`.

<a name="multiple-php-versions"></a>

### Múltiples versiones de PHP

> {nota} esta característica es compatible sólo con *Nginx*.

*Homestead* 6 soporta múltiples versiones de *PHP* en la misma máquina virtual. Puede especificar qué version de *PHP* se usa para un determinado *sitio* en el fichero `Homestead.yaml`. Las versiones de PHP disponibles son: "5.6", "7.0", "7.1" y "7.2":

    sites:
    
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
          php: "5.6"
    

Además, puede utilizar cualquiera de las versiones soportadas usando la línea de comandos (CLI):

    php5.6 artisan list
    php7.0 artisan list
    php7.1 artisan list
    php7.2 artisan list
    

<a name="network-interfaces"></a>

## Interfaces de red

La propiedad `networks` de `Homestead.yaml` configura los interfaces de red para su entorno *Homestead*. Puede configurar tantas interfaces como sean necesarias:

    networks:
    
        - type: "private_network"
          ip: "192.168.10.20"
    

Para activar una interfaz [enlazada](https://www.vagrantup.com/docs/networking/public_network.html), configure una configuración de `bridge` y cambie el tipo red a `public_network`:

    networks:
    
        - type: "public_network"
          ip: "192.168.10.20"
          bridge: "en1: Wi-Fi (AirPort)"
    

Para activar [DHCP](https://www.vagrantup.com/docs/networking/public_network.html), sólo elimine la opción `ip` de su configuración:

    networks:
    
        - type: "public_network"
          bridge: "en1: Wi-Fi (AirPort)"
    

<a name="updating-homestead"></a>

## Actualizando *Homestead*

Puede actualizar *Homestead* en dos simples pasos. Primero, debería de actualizar la caja *Vagrant* utilizando el comando `vagrant box update`:

    vagrant box update
    

A continuación, necesita actualizar el código fuente de *Homestead*. Si clonó el repositorio, puede simplemente ejecutar `git pull origin master` en el mismo sitio donde clonó originalmente el repositorio.

Si tiene instalado *Homestead* a través de su fichero `composer.json` del proyecto, debe asegurarse que éste contiene `"laravel/homestead": "^6"`, y actualizar sus dependencias:

    composer update
    

<a name="old-versions"></a>

## Versiones antiguas

> {tip} Si necesita una versión antigua de PHP compruebe la documentación de [multiples versiones PHP](#multiple-php-versions) antes de intentar utilizar una versión antigua de *Homestead*.

Puede sobreescribir fácilmente la versión del *box* que utiliza *Homestead* agregando la siguiente línea al fichero `Homestead.yaml`:

    version: 0.6.0
    

Un ejemplo:

    box: laravel/homestead
    version: 0.6.0
    ip: "192.168.10.10"
    memory: 2048
    cpus: 4
    provider: virtualbox
    

Cuando utiliza una versión antigua de la caja de *Homestead* necesita combinarla con una versión compatible del código fuente de *Homestead*. A continuación se muestra una tabla que muestra las versiones compatibles de la caja, qué versión del código fuente de *Homestead* usar y la versión de PHP proporcionada:

|             | Versiones *Homestead* | Versiones *Box* |
| ----------- | --------------------- | --------------- |
| PHP 7.0     | 3.1.0                 | 0.6.0           |
| PHP 7.1     | 4.0.0                 | 1.0.0           |
| PHP 7.1     | 5.0.0                 | 2.0.0           |
| PHP 7.1     | 6.0.0                 | 3.0.0           |
| PHP 7.2 RC3 | 6.4.0                 | 4.0.0           |

<a name="provider-specific-settings"></a>

## Configuraciones específicas de proveedor

<a name="provider-specific-virtualbox"></a>

### VirtualBox

Por defecto, *Homestead* establece la configuración `natdnshostresolver` a `on`. Esto permite a *Homestead* utilizar las configuración del DNS de su sistema operativo anfitrión. Si quisiera sobreeescribir este comportamiento, agregue las siguientes líneas a su fichero `Homestead.yaml`:

    provider: virtualbox
    natdnshostresolver: off