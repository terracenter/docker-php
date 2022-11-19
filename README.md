****# Configuración de PHP, PHP-FPM y NGINX para el desarrollo local en Docker

**Fuente:**
https://www.pascallandau.com/blog/php-php-fpm-and-nginx-on-docker-in-windows-10/

Publicado por [Pascal Landau](https://www.pascallandau.com/blog/php-php-fpm-and-nginx-on-docker-in-windows-10/#) el 2018-07-08 22:00:00

Contacto con el autor original:

* https://twitter.com/PascalLandau
* https://www.linkedin.com/in/pascallandau/
* https://github.com/paslandau/
* https://www.youtube.com/channel/UC8hVNCGAtz1DvOOpSQ3Nfzw

Nota: Es la misma información que esta en la fuente, solo que es llevada a Español (traductores en linea) y Linux como sistema operatvo, mucho mas siplificada.

## Configuración del contenedor con PHP cli

**Espacio de trabajo**
`mkdir docker-php`

Sitio donde estaran las aplicaciones
`mkdir app`

Nota:  Si no se crea la carpeta app, y se monta el volumen directamente en el contenedor, la carpeta app se crea automaticamente y toma los permisos root.

Para hacer todo mas simple, se usaran las imagenes oficiales
`docker run -di --name docker-php -v "$PWD"/app:/var/www php:8.1-cli`


| Comando / opciones | Explicación                                                                                 |
| -------------------- | ---------------------------------------------------------------------------------------------- |
| dokcer run         | Inicia el contenedor                                                                         |
| -d                 | Para iniciar un contenedor en modo independiente                                             |
| -i                 | Mantener abierto STDIN aunque no esté conectado. (Mantiene vivo el contenedor               |
| --name             | El nombre del contenedor                                                                     |
| -v                 | Puntos de montaje, usar un carpeta local, sincronizada con una carpeta dentro del contenedor |
| php:8.1-cli        | Imagen oficial de PHP 8.1 Cli                                                                |

El resultado se parece a esto:

```
$ docker run -di --name docker-php -v "dockerrun−di−−namedocker−php−v"$PWD"/app:/var/www php:8.1-cli   
Unable to find image 'php:8.1-cli' locally
8.1-cli: Pulling from library/php
a603fa5e3b41: Already exists
c428f1a49423: Already exists
156740b07ef8: Already exists
fb5a4c8af82f: Already exists
8a8cfb3afb77: Pull complete
f03497c6c1d5: Pull complete
aa773beb2aa3: Pull complete
a84428a91241: Pull complete
fd6a8ec83daa: Pull complete
Digest: sha256:661fd5a2877bfac903851d4d6eb4b02dc43960f36d9353eb1adefed1cf3011d2
Status: Downloaded newer image for php:8.1-cli
e269ba683e043bf6b7c815ceaf09ee882b7cf053b098edd29facb38e369388fc
```

Como no tenemos la imagen en nuestra máquina, (ver `Unable to find image php:8.1-cli locally`), Docker intenta recueprarla del registro oficial en https://hub.docker.com/.

Hemos elegido específicamente la versión "8.1-cli" de la imagen de PHP (lo que significa: solo PHP 8.1 CLI).

Consulte https://hub.docker.com/_/php/ para obtener una lista de todas las etiquetas/imágenes disponibles.

Validamos que el contenedor este en ejeción (Recuerde que para eso se uso la opción **-i**)

```
$ docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS     NAMES
8c0d4c16347d   php:8.1-cli   "docker-php-entrypoi…"   7 seconds ago   Up 6 seconds             docker-php
```

Ahora entramos al contenedor

`docker exec -it docker-php bash`

Validados la versión del PHP

```
# php -v
PHP 8.1.12 (cli) (built: Nov 15 2022 04:42:55) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.1.12, Copyright (c) Zend Technologies
```

Verificar que PHP pueda ejeuctar codigo desde la carpeta app en el host

Salir del contenedor
`exit`

Hola Mundo PHP
`$ echo  '<?php echo "Hello World (php)\n"; ?>' > $(pwd)/app/hello_world.php`

Ahora probar dentro del contenedor

`# php /var/www/hello-world.php `
`Hello World (php)`

# Instalación de Xdebug en el contenedor de PHP

`pecl install xdebug`

Salida del comando:

```
Build process completed successfully
Installing '/usr/local/lib/php/extensions/no-debug-non-zts-20210902/xdebug.so'
install ok: channel://pecl.php.net/xdebug-3.1.6
configuration option "php_ini" is not set to php.ini location
You should add "zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20210902/xdebug.so" to php.ini
```

La extensión xdebug ha sido compilada y se ubica en:
`/usr/local/lib/php/extensions/no-debug-non-zts-20210902/xdebug.so`

Activar Xdebug en el contenedor
`docker-php-ext-enable xdebug`

El comando anterior, colocó el archivo `docker-php-ext-xdebug.ini` en el directorio de contenidos adicionales para el archivo `php.ini`

Para ver la ubicación de la carpeta de contenidos adicionales para el archivo php.ini

```
# php -i | grep "additional .ini"
Scan this dir for additional .ini files => /usr/local/etc/php/conf.d

# ls -l /usr/local/etc/php/conf.d
total 8
-rw-r--r-- 1 root root 17 Nov 15 04:44 docker-php-ext-sodium.ini
-rw-r--r-- 1 root root 22 Nov 18 20:19 docker-php-ext-xdebug.ini
```

Validar que está activado el xdebug
`php -m | grep xdebug`

Salir y detener el contenedor

```
exit
$ docker stop docker-php
docker-php
```

NOTA: Todos los cambios que se realizaron en el contendor docker-php estan dispoibles, ya que solo de detuvo el contenedor, pero si se llega a borrar o a reconstruir todo esos cambios se perderan.

Ejemplo de perdida de los cambios al borrar el contenedor:

```
docker rm docker-php
docker-php
```

Confirma perdida de los cambios
`docker run -di --name docker-php -v "$PWD"/app:/var/www php:8.1-cli`

```
docker exec -it docker-php bash
php -m | grep xdebug
```

Ya no hay nada.

Borramos docker-php, para no tener problemas con el nombre
`docker rm -f docker-php`

## Creacion de una imagen e instalar paquetes con un Dockerfile

```
mkdir php-cli
touch php-cli/Dockerfile
```

Contenido del Dockerfile

```
FROM php:8.1-cli
RUN pecl install xdebug
&& docker-php-ext-enable xdebug
```

Creación de la imagen
`docker build -t docker-php-image -f Dockerfile .`

Nota: -f es opcional, ya que el valor predeterminado es Dockerfile

Verificar que xdebug esta instalado y activo
`docker run -di --name docker-php -v "$PWD"/../app:/var/www docker-php-imag`

Nota: "$PWD"**/../**, se refiere a un directorio superior, es la carpetas de las aplicaciones.

`$ docker exec -it docker-php bash`

```
# php -m | grep xdebug
xdebug
```

Tambien se puede validar sin tener que entrar al contenedor, con el siguiente comando:

```
docker run docker-php-image php -m | grep xdebug
xdebug 
```

Nota: Es buena idea para e elimiar todos los contenedores

```
$ docker rm -f dockerrm−f$(docker ps -aq)
3da7fef90065
a030b66d194f
```

Y subimos al directorio principal del proyecto
`cd ..`

## Configuración del servidor web Nginx con php-fpm

Se usara la imagen oficial de nginx, se instalaran unos paquetes y se ubicaran los archivos de configuración del nginx

Ejecución del contenedor nginx la ultima version con el nombre docker-nix
`docker run -di --name docker-nginx -p 80:80 nginx:latest`

Nota: **-p** indica que se use el prto 80 en el host conectado al puerto 80 del contenedor

Ahora verificamos la información que requerimos
`docker exec -it docker-nginx bash`

```
# nginx -v
nginx version: nginx/1.23.2
built by gcc 10.2.1 20210110 (Debian 10.2.1-6) 
built with OpenSSL 1.1.1n  15 Mar 2022
TLS SNI support enabled
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-g -O2 -ffile-prefix-map=/data/builder/debuild/nginx-1.23.2/debian/debuild-base/nginx-1.23.2=. -fstack-protector-strong -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -Wl,--as-needed -pie'
```

El archivo de confguracion está en la ruta:
--conf-path=**/etc/nginx/nginx.conf**

Se debe tener en cuenta los includes en el archivo de configuración, sobre todo `include /etc/nginx/conf.d/*.conf`

```
# cat /etc/nginx/nginx.conf | grep include
include       /etc/nginx/mime.types;
include /etc/nginx/conf.d/*.conf;
```

Ya que la configuracion predetermida de nginx es **default.conf**

```
# ls -alh /etc/nginx/conf.d/
total 20K
drwxr-xr-x 1 root root 4.0K Nov 18 21:16 .
drwxr-xr-x 1 root root 4.0K Nov 15 13:14 ..
-rw-r--r-- 1 root root 1.1K Nov 18 21:16 default.conf
```

Probamos que el Nginx exte funcionando en el host
`exit`

y el nevgador

`127.0.0.1`

```
Welcome to nginx!
If you see this page, the nginx web server is successfully installed and working. Further configuration is required.For online documentation and support please refer to nginx.org.
Commercial support is available at nginx.com.Thank you for using nginx.
```

Cambiar la configración de la raiz de Nginx, a /var/www y ver si funciona bien.
`$ docker exec -it docker-nginx bash`

```
# sed -i "s#/usr/share/nginx/html#/var/www#" /etc/nginx/conf.d/default.conf
# mkdir -p /var/www
# echo "Hello world!" > /var/www/index.html

```

Recargar la configuración e nginx
`nginx -s reload`

Revisar en el nevegador
**Hello world!**

Salimos, de cerramos el contenedor para continuar

```
# exit
$ docker rm -f dockerrm−f$(docker ps -aq)
```

La imagen de de nginx se construirá con la siguiente estructura de carpetas:

+ docker-php
  + nginx
    + conf.d\
      + site.conf
    + Dockerfile
  + app\
    + index.html
    + hello-world.php

Contenidos de los archivos:
nginx\Dockerfile
`FROM nginx:latest`

```
$ mkdir -p nginx/conf.d
$ mkdir−pnginx/conf.d
$ touch nginx/conf.d/site.conf  
```

Hora a contruir la imagen de nginx

```
$ cd nginx
$ docker build -t docker-nginx-image .
```

Ejecutamos contenedor nginx con los volumenes de la configuracion
`$ docker run -di --name docker-nginx -p 80:80 -v "$PWD"/config.d:/etc/nginx/conf.d/  -v "PWD"/config.d:/etc/nginx/conf.d/−v"$PWD"/../app:/var/www docker-nginx-image`

**Nota**: Recuerda `/../` se debe a que la carpeta app esta en un directorio superior, y que `-v` es para indicar los volumnes que se montan en el host que se sincronizan con el contenedor

En el navegador en la ruta http://127.0.0.1/ debe se salir:
**Hello World**

docker stop docker-nginx

## Configurando php-fpm

Ya estamos familiarizados con la imagen PHP oficial de docker, pero hasta ahora solo hemos usado la versión cli-only. Los de FPM se pueden extraer usando las etiquetas -fpm (por ejemplo, como php:8.1-fpm). Al igual que con nginx, primero exploremos la imagen de php-fpm:

`$ docker run -di --name php-fpm-test php:8.1-fpm`

Vamos a buscar la información que requerimos, simplificando las cosas

`$ docker exec -it php-fpm-test bash`

```
#  php-fpm -i | grep config
Configure Command =>  './configure'  '--build=x86_64-linux-gnu' '--with-config-file-path=/usr/local/etc/php' '--with-config-file-scan-dir=/usr/local/etc/php/conf.d' '--enable-option-checking=fatal' '--with-mhash' '--with-pic' '--enable-ftp' '--enable-mbstring' '--enable-mysqlnd' '--with-password-argon2' '--with-sodium=shared' '--with-pdo-sqlite=/usr' '--with-sqlite3=/usr' '--with-curl' '--with-iconv' '--with-openssl' '--with-readline' '--with-zlib' '--disable-phpdbg' '--with-pear' '--with-libdir=lib/x86_64-linux-gnu' '--disable-cgi' '--enable-fpm' '--with-fpm-user=www-data' '--with-fpm-group=www-data' 'build_alias=x86_64-linux-gnu'
fpm.config => no value => no value
Openssl default config => /usr/lib/ssl/openssl.cnf
PHPIZE_DEPS => autoconf                 dpkg-dev                file            g++             gcc             libc-dev                make            pkg-config      re2c
$_SERVER['PHPIZE_DEPS'] => autoconf             dpkg-dev                file            g++             gcc             libc-dev                make            pkg-config               re2c
$_ENV['PHPIZE_DEPS'] => autoconf                dpkg-dev                file            g++             gcc             libc-dev                make            pkg-config               re2c

```

Uno de los que se necesita es:
--with-config-file-path=`/usr/local/etc/php`

```
# grep "include=" /usr/local/etc/php-fpm.conf
include=etc/php-fpm.d/*.conf
```

Como el include muestra es una ruta relativa:
`etc/php-fpm.d/*.conf`

Se valida con:

```
# grep -C 6 "include=" /usr/local/etc/php-fpm.conf
; Include one or more files. If glob(3) exists, it is used to include a bunch of
; files from a glob(3) pattern. This directive can be used everywhere in the
; file.
; Relative path can also be used. They will be prefixed by:
;  - the global prefix if it's been set (-p argument)
;  - /usr/local otherwise
include=etc/php-fpm.d/*.conf
```

Entonces esa ruta relativa hace referencia `/usr/local` y la rupta absoluta queda `/usr/local/etc/php-fpm.d/*.conf` por tanto al ejecutar

```
# cat /usr/local/etc/php-fpm.d/www.conf

[....]
; The address on which to accept FastCGI requests.
; Valid syntaxes are:
;   'ip.add.re.ss:port'    - to listen on a TCP socket to a specific IPv4 address on
;                            a specific port;
;   '[ip:6:addr:ess]:port' - to listen on a TCP socket to a specific IPv6 address on
;                            a specific port;
;   'port'                 - to listen on a TCP socket to all addresses
;                            (IPv6 and IPv4-mapped) on a specific port;
;   '/path/to/unix/socket' - to listen on a unix socket.
; Note: This value is mandatory.
listen = 127.0.0.1:9000
```

Salir del contenedor y subir al directrio principal del proyecto

```
# exit
docker rm -f php-fpm-test
$ cd
```

Instalando el xdebug, usando el Dockerfile

+ docker-php
  + nginx\
    + conf.d\
      + site.conf
    + Dockerfile
  + app\
    + index.html
    + hello-world.php
  + php-fpm\
    + Dockerfile

```
mkdir php-fpm
touch php-fpm/Dockerfile
cd php-fpm
```

Contenido del php-fpm\Dockerfile

```
FROM php:8.1-fpm
RUN pecl install xdebug && docker-php-ext-enable xdebug
```

Creamos la imagen docker-php-fpm-image
`docker build -t docker-php-fpm-image .`

## Conectando nginx y php-fpm

Primero regresamos al directorio principal del proyecto
`cd`

vamos a ver las redes que estan diponlbles

```
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
e646973e7cd2   bridge    bridge    local
b8bb46a758fd   host      host      local
bd78cbc4704e   none      null      local
```

Podemos usar la bridge o creamos una nueva
`$docker network create --driver bridge web-network`

Donde:

* create, es para crear la nueva red
* --dirvers, es el tipo de red
* web-network, es el nombre de la red

Iniciamos el contenedor
`$ docker run -di --name docker-nginx -p 80:80 -v "$PWD"/nginx/conf.d:/etc/nginx/conf.d/ -v "$PWD"/app:/var/www docker-nginx-image`

Ahora conectados el contenedor docker-nginx a la red web-network
`$ docker network connect web-network docker-nginx`

Ahora ejecutamos el contenedor docker-php-fpm, con el volumen de la aplicacion app y lo conectadomos a la red web-network

`docker run -di --name docker-php-fpm -v "$PWD"/app:/var/www --network web-network docker-php-fpm-image`

vamos a revisar que los dos contenedores esten en la misma red

```
$ docker network inspect web-network
[....]
"Name": "docker-php-fpm",
"IPv4Address": "172.18.0.3/16",
"Name": "docker-nginx"
"IPv4Address": "172.18.0.2/16",
```

Se configura Nnginx paara que acepte todas las solicitudes relacionadas con PHP a php-fpm cambiando el `nginx\conf.d\site.conf`

```
server {
listen      80;
server_name localhost;
root        /var/www;location ~ \.php$ {
try_files KaTeX parse error: Expected '}', got 'EOF' at end of input:  {
try_files $uri =404;
fastcgi_pass docker-php-fpm:9000;
include fastcgi_params;
fastcgi_param  SCRIPT_FILENAME $document_rootdocumentroot$fastcgi_script_name;
    }
}
```

Se recarga la configuracion del Nginx

```
$ docker exec -it docker-nginx bash
# nginx -s reload# exit
# exit
```

Se valida que este funcionando `http://127.0.0.1/hello-world.php`

Hello World (php)
