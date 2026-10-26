# Manual de preparación del servidor web Ubuntu (YRM-USED)
*(Servidor de desarrollo para aplicaciones web basadas en el uso de PHP)*

---

## Contenido de la guía

- [Manual de preparación del servidor web Ubuntu (YRM-USED)](#manual-de-preparación-del-servidor-web-ubuntu-yrm-used)
  - [Índice de Contenidos](#índice-de-contenidos)
  - [1. Configuración Inicial](#1-configuración-inicial)
    - [1.1. Preparación e Instalación del SO](#11-preparación-e-instalación-del-so)
    - [1.2. Configuración de Red y Cambio de nombre](#12-configuración-de-red-y-cambio-de-nombre)
    - [1.3. Actualización, Fecha, SSH, Firewall y Antivirus](#13-actualización-fecha-ssh-firewall-y-antivirus)
    - [1.4. Comprobación del Sistema y Conexión](#14-comprobación-del-sistema-y-conexión)
  - [2. Cuentas de Administración](#2-cuentas-de-administración)
    - [2.1. Cuentas Administradoras (`miadmin`, `miadmin2`)](#21-cuentas-administradoras-miadmin-miadmin2)
  - [3. Apache2 (Servidor Web)](#3-apache2-servidor-web)
    - [3.1. Instalación y Configuración Básica](#31-instalación-y-configuración-básica)
    - [3.2. Redirección ErrorLog y ErrorDocument (404)](#32-redirección-errorlog-y-errordocument-404)
    - [3.3. Habilitar .htaccess (AllowOverride)](#33-habilitar-htaccess-allowoverride)
    - [3.4. Configuración HTTPS (Certificado SSL YRM)](#34-configuración-https-certificado-ssl-mdc)
    - [3.5. Redirección de HTTP a HTTPS](#35-redirección-de-http-a-https)
  - [4. Cuenta de Desarrollo (Operador Web)](#4-cuenta-de-desarrollo-operador-web)
    - [4.1. Creación del Usuario y Permisos Web](#41-creación-del-usuario-y-permisos-web)
  - [5. Ejecución PHP con PHP-FPM](#5-ejecución-php-con-php-fpm)
    - [5.1. Instalación de PHP-FPM](#51-instalación-de-php-fpm)
    - [5.2. Integración de PHP-FPM con Apache2](#52-integración-de-php-fpm-con-apache2)
    - [5.3. Configuración de php.ini para Desarrollo](#53-configuración-de-phpini-para-desarrollo)
  - [6. MariaDB (Servidor de Base de Datos)](#6-mariadb-servidor-de-base-de-datos)
    - [6.1. Instalación y Acceso Remoto](#61-instalación-y-acceso-remoto)
    - [6.2. Creación de Usuario Administrador](#62-creación-de-usuario-administrador)
    - [6.3. Asistente de Seguridad](#63-asistente-de-seguridad)
  - [7. phpMyAdmin](#7-phpmyadmin)
  - [8. XDebug](#8-xdebug)
  - [9. Sitios Virtuales y DirectoryIndex](#9-sitios-virtuales-y-directoryindex)
  - [10. SFTP (Usuarios Enjaulados)](#10-sftp-usuarios-enjaulados)
  - [11. Herramientas de Desarrollo (phpDocumentor)](#11-herramientas-de-desarrollo-phpdocumentor)

---

## 1. Preparación inicial del servidor

En este apartado se prepara Ubuntu, se configura la conexión de red y se dejan instaladas las herramientas básicas para administrar el equipo.

### 1.1. Preparar la máquina e instalar Ubuntu

Se definen los recursos de la máquina virtual y se comprueba cómo ha quedado instalado el sistema.

*   **Nombre de la máquina:** `YRM-USED`
*   **RAM:** `2048 MB` (2 GB)
*   **Particiones:** `150GB` (`/`), `350GB` (`/var`) y `4GB` de `swap`.

Para consultar la versión de Ubuntu, el kernel, los discos y el espacio disponible:

    uname -a
    lsb_release -a
    lsblk
    df -h
    lsblk -fm
    lsblk -a
    fdisk -l

### 1.2. Ajustes de red y nombre del equipo

Se asigna un nombre al servidor y una dirección IP adecuada para trabajar desde clase o desde casa.

Asegúrate de que el nombre del host sea correcto:

    sudo hostnamectl set-hostname YRM-USED
    sudo nano /etc/hosts

Para configurar la red estática editamos Netplan:

    cd /etc/netplan
    ls
    sudo cp 00-installer-config.yaml 00-installer-config.yaml.backup
    sudo nano 00-installer-config.yaml

**Parámetros de Red en CLASE (IES Los Sauces)**:

    network:
      ethernets:
        enp0s3:
          addresses:
           - 10.199.11.90/22
          nameservers:
             addresses:
             - 10.151.123.21
             - 10.151.126.21
             search: [educa.jcyl.es]
          routes:
              - to: default
                via: 10.199.8.1
      version: 2

**Parámetros de Red en CASA**:

    network:
      ethernets:
        enp0s3:
          addresses:
           - 192.168.1.100/24
          nameservers:
             addresses:
             - 8.8.8.8
             - 8.8.4.4
          routes:
              - to: default
                via: 192.168.1.1
      version: 2

Aplica la configuración de red seleccionada: 

    sudo netplan apply

Para ver las direcciones IP, la ruta de salida y la configuración DNS:

    ip a
    ip r
    resolvectl status

### 1.3. Actualizaciones, acceso remoto y seguridad básica

Se actualiza Ubuntu, se configura la hora y se habilitan servicios básicos de acceso y protección.

Actualización del sistema y configuración de zona horaria:

    sudo apt update && sudo apt upgrade -y
    date
    sudo timedatectl set-timezone Europe/Madrid

Habilitar Cortafuegos y puertos básicos (SSH):

    sudo ufw enable
    sudo ufw allow 22/tcp

Instalar y actualizar Antivirus ClamAV:

    sudo apt install clamav clamav-daemon -y
    sudo systemctl stop clamav-freshclam
    sudo freshclam
    sudo systemctl start clamav-freshclam
    sudo systemctl enable clamav-freshclam

### 1.4. Revisar el sistema y probar el acceso

Se comprueba que el servidor responde por red y que permite entrar mediante SSH.

Desde el terminal (CMD) de tu máquina anfitriona, haz ping y conéctate:

    ping 10.199.11.90
    ssh miadmin@10.199.11.90

---

## 2. Usuarios con permisos de administración

Las cuentas administrativas permiten realizar tareas del sistema sin utilizar siempre el usuario principal.

### 2.1. Crear una segunda cuenta administrativa

Se crea un usuario adicional con permisos para administrar el servidor.

Crear `miadmin2` y añadirlo a varios grupos administrativos:

    sudo useradd -m -G sudo,adm,cdrom,dip,plugdev,lxd -s /bin/bash miadmin2
    sudo passwd miadmin2

Comprobar pertenencia a grupos:

    cat /etc/group | grep miadmin
    cat /etc/passwd | grep miadmin

---

## 3. Apache2: puesta en marcha del servidor web

Apache recibe las peticiones del navegador y sirve las páginas alojadas en el servidor.

### 3.1. Instalar Apache y comprobar el servicio

Se instala Apache, se abre el puerto HTTP y se verifica que el servicio está funcionando.

Instalación de Apache y apertura del puerto 80:

    sudo apt update
    sudo apt install apache2 -y
    sudo ufw allow 80
    sudo systemctl status apache2

### 3.2. Registro de errores y página para errores 404

Se indica dónde guardar los errores y qué página mostrar cuando no se encuentra un recurso.

Creamos un directorio para los errores en la raíz web:

    sudo mkdir /var/www/html/error
    sudo touch /var/www/html/error/error.log

Editamos el VirtualHost por defecto (`000-default.conf`):

    cd /etc/apache2/sites-available
    sudo cp 000-default.conf 000-default.conf.backup
    sudo nano 000-default.conf

Dentro de `<VirtualHost *:80>`, añade o modifica:

    ErrorLog /var/www/html/error/error.log
    ErrorDocument 404 /error/404.html

### 3.3. Permitir reglas propias con AllowOverride

Esta opción permite que los archivos `.htaccess` definan determinadas reglas para el directorio web.

Usar el comando `sudo a2enmod rewrite` para activar el modulo de rewrite   

Entrar al archivo de configuracion usando `sudo nano /etc/apache2/apache2.conf`  

**Modificar el valor de AllowOverride en el archivo:**
```bash
<Directory /var/www/>
  ...
  AllowOverride All
</Directory>

ServerName yrm-used
```

Comprobación de la sintaxis de la configuración: `sudo apache2ctl configtest`

Recarga la configuración sin detener completamente Apache: `sudo systemctl reload apache2`

Reinicia Apache para aplicar todos los cambios: `sudo systemctl restart apache2`

### 3.4. Activar HTTPS con un certificado autofirmado

HTTPS cifra la comunicación entre el navegador y el servidor; el certificado creado aquí es para pruebas y puede generar un aviso en el navegador.

Generar un certificado SSL almacena información básica sobre un sitio y lo acompaña de un archivo de claves para manejar datos cifrados de forma segura. Generaremos la clave privada y el CSR (Certificate Signing Request) que garantiza la integridad y autenticación.

Activamos el módulo SSL de Apache, que se basa en OpenSSL para proporcionar el modo de criptografía:

    sudo a2enmod ssl

Comando para crear los archivos del certificado:

    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt

Información solicitada por el sistema (CSR):
*   **Country Name:** `ES`
*   **State or Province Name:** `ZAMORA`
*   **Locality Name:** `BENAVENTE`
*   **Organization Name:** `IES LOS SAUCES` (Preferente en mayúsculas).
*   **Organizational Unit Name:** `INFORMATICA`
*   **Common Name:** `yrm-used` (Nombre del dominio/servidor sin espacios).
*   **Email Address:** (Dejar en blanco).

Configuración del sitio virtual seguro:

    cd /etc/apache2/sites-available/
    sudo cp default-ssl.conf default-ssl.conf.backup
    sudo nano default-ssl.conf

Configuración dentro de `<VirtualHost *:443>`:

    ServerAdmin webmaster@localhost
    ServerName 10.199.11.90
    DocumentRoot /var/www/html
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
    SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
    ErrorLog ${APACHE_LOG_DIR}/mdc-ssl-error.log
    CustomLog ${APACHE_LOG_DIR}/mdc-ssl-access.log combined

Activación del sitio virtual seguro:

    sudo a2ensite default-ssl.conf
    sudo ufw allow 443
    sudo systemctl restart apache2

Comprobación desde el navegador estableciendo una conexión segura: `https://10.199.11.90`.

### 3.5. Enviar las peticiones HTTP hacia HTTPS

Se configura Apache para que las visitas por HTTP se redirijan a la versión segura de la web.

Para activar este servicio se tiene que hacer con el operadorweb desde el CMD o MobaXterm.

Activamos el módulo rewrite, necesario para redirigir de un sitio a otro:

    sudo a2enmod rewrite

En el archivo `.htaccess` (previamente activado), añadimos las directivas de redirección:

    sudo nano /var/www/html/.htaccess

Añade lo siguiente:

    RewriteEngine On
    RewriteCond %{SERVER_PORT} 80
    RewriteRule ^(.*)$ https://10.199.11.90/$1 [R,L]

*Explicación de directivas:* `RewriteEngine On` habilita el módulo. `RewriteCond %{SERVER_PORT} 80` indica que todas las peticiones al puerto 80 pasen por la regla especificada. `[R,L]` realiza la redirección y deshabilita que reglas de reescritura posteriores afecten a la URL.

Reinicia y testea:

    sudo apache2ctl configtest
    sudo systemctl reload apache2

---

## 4. PHP-FPM: ejecutar PHP desde Apache

PHP-FPM ejecuta los scripts PHP y permite que Apache se comunique con el intérprete mediante FastCGI.

PHP-FPM permite que Apache procese los archivos PHP mediante un servicio específico, en lugar de tratarlos como archivos estáticos.

### 4.1. Instalar el servicio PHP-FPM

Se instala el servicio que ejecutará los archivos PHP del proyecto.

Se instala la versión de PHP-FPM utilizada en este servidor:

    sudo apt install php8.5-fpm -y

### 4.2. Conectar PHP-FPM con Apache

Se activan los módulos y la configuración necesarios para enlazar ambos servicios.

Se activan los módulos necesarios para que Apache pueda comunicarse con PHP-FPM:

    sudo a2enmod proxy_fcgi setenvif

Se habilita la configuración correspondiente a PHP 8.5:

    sudo a2enconf php8.5-fpm

Se reinician los servicios para aplicar los cambios:

    sudo systemctl restart php8.5-fpm
    sudo systemctl restart apache2

### 4.3. Ajustar PHP para trabajar en desarrollo

Se modifican opciones de PHP para facilitar las pruebas, la carga de archivos y la detección de errores.


Hacer copia de seguridad y editar:

    cd /etc/php/8.5/fpm/
    sudo cp php.ini php.ini.bk2026
    sudo nano php.ini

Modifica las siguientes directivas para tu entorno de desarrollo:

    memory_limit = 256M
    upload_max_filesize = 100M
    max_execution_time = 360
    date.timezone = Europe/Madrid
    display_errors = On
    error_reporting = E_ALL
    display_startup_errors = On
    log_errors = On
    error_log = /var/log/php_errors.log

Aplica los cambios: 
    
    sudo systemctl restart php8.5-fpm.service

---

## 5. Cuenta de trabajo web: operadorweb

Se crea una cuenta destinada a gestionar los archivos de la web después de dejar PHP instalado y configurado.

Esta cuenta se utiliza para gestionar los archivos de la web sin trabajar directamente con la cuenta de administración del sistema. Se crea después de instalar y configurar PHP, tal como se solicita en el orden de instalación.

### 5.1. Crear el usuario y asignar permisos sobre la web

Se asigna el usuario al grupo web y se dan permisos para trabajar con los archivos alojados en Apache.

Se crea `operadorweb` sin directorio personal propio (`-M`), usando `/var/www/html` como directorio de trabajo y el grupo `www-data`:

    sudo useradd -M -d /var/www/html -N -g www-data -s /bin/bash operadorweb
    sudo passwd operadorweb

Se asigna la propiedad de los archivos web al usuario y al grupo correspondiente. El permiso `775` permite al propietario y al grupo leer, modificar y acceder a los archivos:

    sudo chown -R operadorweb:www-data /var/www/html
    sudo chmod -R 775 /var/www/html

---

## 6. MariaDB: instalación y configuración de la base de datos

MariaDB almacena los datos que utilizan las aplicaciones web.

### 6.1. Instalar MariaDB y habilitar acceso remoto

Se instala el servidor de bases de datos y, si hace falta, se permite la conexión desde otros equipos.

Instalación:

    sudo apt install mariadb-server -y

Para permitir conexiones remotas:

    sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

Cambia `bind-address = 127.0.0.1` a `bind-address = 0.0.0.0` y reinicia:

    sudo systemctl restart mariadb
    sudo ufw allow 3306/tcp

### 6.2. Crear una cuenta administradora de SQL

Se crea un usuario con permisos administrativos para gestionar las bases de datos.

Crearemos tu usuario `adminsql` ya que root funciona por *unix_socket*:

    sudo mariadb

Dentro de SQL ejecuta:

    CREATE USER 'adminsql'@'%' IDENTIFIED BY 'paso';
    GRANT ALL PRIVILEGES ON *.* TO 'adminsql'@'%' WITH GRANT OPTION;
    FLUSH PRIVILEGES;
    EXIT;

### 6.3. Aplicar los ajustes básicos de seguridad

Se ejecuta el asistente que ayuda a retirar opciones inseguras de la instalación inicial.

Asegura el servidor ejecutando este script y siguiendo las instrucciones en pantalla:

    sudo mariadb-secure-installation

---

## 7. phpMyAdmin: administrar bases de datos desde el navegador

phpMyAdmin ofrece una interfaz web para consultar y gestionar las bases de datos de MariaDB.

Antes de instalar, vemos qué módulos tenemos para compararlos después:

    php -m > /home/miadmin/listadomodulos.txt

Instalamos, seleccionamos *Apache* y confirmamos la base de datos `dbconfig-common`:

    sudo apt install phpmyadmin -y
    php -m > /home/miadmin/listadomodulos2.txt
    diff /home/miadmin/listadomodulos.txt /home/miadmin/listadomodulos2.txt

Configuramos el acceso en Apache creando un enlace simbólico:

    sudo ln -sf /etc/phpmyadmin/apache.conf /etc/apache2/conf-available/phpmyadmin.conf
    sudo a2enconf phpmyadmin
    sudo systemctl restart apache2

---

## 8. Xdebug: depurar código PHP paso a paso

Xdebug es una extensión de PHP que permite encontrar errores y seguir la ejecución del programa con más detalle. También incluye herramientas para analizar el rendimiento.

**Funciones principales:**
- Depuración paso a paso (*step debugging*).
- Puntos de interrupción (*breakpoints*) para pausar el programa en una línea concreta.
- Inspección de variables y de la secuencia de llamadas.
- Integración con editores como NetBeans, VS Code o PhpStorm.
- Comunicación con el IDE mediante el protocolo DBGp, que utiliza normalmente el puerto `9003`.

### 8.1. Comprobar si Xdebug está instalado

Se consulta la versión de PHP y se filtran las líneas relacionadas con Xdebug:

    sudo php -v | grep -i xdebug

Si no aparece ninguna línea, se instala el paquete correspondiente a la versión de PHP usada en esta guía:

    sudo apt update
    sudo apt install php8.5-xdebug -y

### 8.2. Configurar Xdebug para el desarrollo

Se abre el archivo de configuración de Xdebug para PHP-FPM:

    sudo nano /etc/php/8.5/fpm/conf.d/20-xdebug.ini

Se añaden estas opciones:

    xdebug.mode=develop,debug
    xdebug.start_with_request=yes
    xdebug.client_host=127.0.0.1
    xdebug.client_port=9003
    xdebug.log_level=7
    xdebug.idekey="netbeans-xdebug"
    xdebug.discover_client_host=1

**Para qué sirve cada opción:**
- `xdebug.mode=develop,debug`: activa las ayudas de desarrollo y la depuración.
- `xdebug.start_with_request=yes`: inicia la depuración en cada petición PHP.
- `xdebug.client_host=127.0.0.1`: indica la dirección del equipo al que Xdebug intentará conectarse. Si el IDE está en otro ordenador, esta dirección tendrá que adaptarse.
- `xdebug.client_port=9003`: establece el puerto de comunicación con el IDE.
- `xdebug.log_level=7`: guarda información detallada para ayudar a detectar problemas de conexión.
- `xdebug.idekey="netbeans-xdebug"`: define una clave identificativa para la sesión del depurador.
- `xdebug.discover_client_host=1`: permite intentar detectar la dirección del cliente a partir de la petición.

Se guardan los cambios y se reinicia PHP-FPM y Apache para aplicar la configuración:

    sudo systemctl restart php8.5-fpm
    sudo systemctl restart apache2

Se puede comprobar que el módulo está cargado con:

    php -m | grep -i xdebug

**Nota:** `127.0.0.1` solo apunta al propio servidor. Si NetBeans se ejecuta en el ordenador anfitrión y PHP en la máquina virtual, habrá que configurar la dirección del equipo anfitrión que sea accesible desde la máquina virtual. La IP `10.199.11.90/22` es la IP del servidor en clase, no necesariamente la dirección del ordenador donde se ejecuta NetBeans.

## 9. Sitios virtuales y página de inicio predeterminada

Los sitios virtuales permiten alojar distintas webs en un mismo servidor y definir qué archivo se carga por defecto.

## 10. SFTP con usuarios limitados a su directorio

SFTP permite transferir archivos de forma cifrada y restringir el acceso de ciertos usuarios.

## 11. Documentación del código con phpDocumentor

phpDocumentor genera documentación a partir de los comentarios y anotaciones incluidos en el código PHP.


Autor: Yolanda Rodríguez

---

## Añadir capturas de pantalla a la guía

Sí, puedes intercalar imágenes entre los pasos para que se vea cómo queda cada configuración. Guarda las capturas en una carpeta junto al archivo Markdown, por ejemplo `imagenes/`, y utiliza una ruta relativa:

```markdown
![Configuración de red en Netplan](imagenes/netplan-red.png)
```

El texto entre corchetes es una descripción de la imagen y sirve también como texto alternativo. El nombre del archivo debe coincidir exactamente, incluidas las mayúsculas y la extensión. Para controlar el tamaño, si el visor Markdown lo permite, puedes usar HTML:

```html
<img src="imagenes/netplan-red.png" alt="Configuración de red en Netplan" width="700">
```

Es recomendable poner cada captura justo después de la explicación o del comando al que corresponde. Antes de entregar la guía, comprueba que las imágenes se ven desde el programa en el que vayas a abrir o exportar el Markdown.
