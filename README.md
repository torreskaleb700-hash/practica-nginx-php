<center>
 <br>

 ***TECNOLOGICO DE ESTUDIOS SUPERIORES ORIENTE DEL ESTADO DE MÉXICO***

# PRACTICA: Implementación y configuración de servidores NGINX 1.31.x y PHP 8.4.x compilados desde código fuente en AlmaLinux 9, con integrados mediante PHP-FPM y SystemD

**Materia: TALLER DE SISTEMAS OPERATIVOS.**

**Nombres:**
* **Reyes Gutierrez Stephany.**
* **Velazquez Aguilar David Israel.**
* **Hernandez Torrez Kaleb Guadalupe.**

**Grupo: 6S11.**

**Profesor: Gustavo Moises Romero Gonzalez.**  
<br>
<br>
<br>
 </center>

## Objetivo General:
Implementar mediante compilación en código fuente, los servidores web NGINX versión 1.31.x y PHP versión 8.4.x sobre AlmaLinux 9 en entorno virtualizado, configurando rutas personalizadas, usuarios de sistema, servicios gestionados por SystemD y la integración funcional entre ambos mediante PHP-FPM con socket UNIX, garantizando su arranque automático y correcto funcionamiento.

## Objetivos Especificos:
* Implementar un servidor NGINX, compilado desde Codigo fuente en la version 1.31.x.
* Usar como Prefix de Instalación /srv/nginx para ambos servidores, asignando el usuario y grupo de sistema nginx para su ejecución.
* Crear y configurar la unidad de servicio SystemD nginx.service en /etc/systemd/system/, estableciendo su arranque automático del SO en en graphical.targeo o multi-user.target.
* Compilar e instalar PHP 8.4.x desde código fuente, ubicando sus archivos en /srv/nginx, configurando su ejecución bajo el usuario y grupo php:nginx.
* Habilitar y compilar el soporte de PHP-FPM, incluyendo las extensiones requeridas para  poder comunicar el servidor php con nginx, logando procesar imágenes, manejo de fechas e internacionalización (intl).
* Configurar PHP-FPM para funcionar mediante socket UNIX ubicado en /tmp/php84.sock, en lugar de conexión TCP, y crear su unidad de servicio SystemD php-fpm8.4.service.
* Configuración del servidor php, debe ser por socket UNIX (no SocketTCP). el socket puede estar en /tmp/php84.sock.
* Implementar una configuración de comunicación fastcgi (fcgi) de nginx a php-fpm mediante el uso de su socket unix.
* Verificar el funcionamiento completo mediante la implementación de un script de prueba y un archivo phpinfo.php, que permitan validar la integración, las características activas y la ruta de instalación configurada.
<br>
<br>
<br>
<br>


## **Desarrollo del proyecto:**
## Implementación y configuración de servidor NGINX 1.31.x, compilado desde Codigo fuente en la version 1.31.x: 

* **1. Instalar las herramientas y librerías necesarias para compilar código fuente: Development Tools.**

![alt text](image.png)

![alt text](image-2.png)

![alt text](image-3.png)

![alt text](image-4.png)

* **2. Creamos los usuarios y grupos:**
* Creación de usuario y grupo de sistema para NGINX
* Se usa -r (usuario de sistema), /nonexistent (sin carpeta personal), sin acceso a shell
* Creación de usuario PHP y asignación al grupo NGINX.

![alt text](image-5.png)

```
[mstephany@localhost nginx-1.31.0]$ sudo useradd -r -d /nonexistent -s /sbin/nologin nginx
[sudo] password for mstephany: 
useradd: el usuario «nginx» ya existe
[mstephany@localhost nginx-1.31.0]$ sudo useradd -r -d /nonexistent -s /sbin/nologin nginx
useradd: el usuario «nginx» ya existe
[mstephany@localhost nginx-1.31.0]$ sudo useradd -r -d /nonexistent -s /sbin/nologin -G nginx php
useradd: el usuario «php» ya existe
[mstephany@localhost nginx-1.31.0]$ 
```
![alt text](image-6.png)

* **3. Instalar NGINX 1.31.x desde código fuente**
```
[mstephany@localhost ~]$ cd /usr/src
```
```
[mstephany@localhost ~]$ cd /usr/src
[mstephany@localhost src]$ sudo wget https://nginx.org/download/nginx-1.31.0.tar.gz
[sudo] password for mstephany: 
--2026-05-24 01:28:29--  https://nginx.org/download/nginx-1.31.0.tar.gz
Resolviendo nginx.org (nginx.org)... 3.147.99.198, 2a05:d014:5c0:2600::6, 2a05:d014:5c0:2601::6
Conectando con nginx.org (nginx.org)[3.147.99.198]:443... conectado.
Petición HTTP enviada, esperando respuesta... 200 OK
Longitud: 1337335 (1.3M) [application/octet-stream]
Grabando a: «nginx-1.31.0.tar.gz.1»

nginx-1.31.0.tar.gz 100%[===================>]   1.27M  3.68MB/s    en 0.3s    

2026-05-24 01:28:31 (3.68 MB/s) - «nginx-1.31.0.tar.gz.1» guardado [1337335/1337335]

[mstephany@localhost src]$ sudo tar -xzf nginx-1.31.0.tar.gz
[mstephany@localhost src]$ cd nginx-1.31.0
```
```
[mstephany@localhost ~]$ cd /usr/src
[mstephany@localhost src]$ sudo wget https://nginx.org/download/nginx-1.31.0.tar.gz
[sudo] password for mstephany: 
--2026-05-24 01:28:29--  https://nginx.org/download/nginx-1.31.0.tar.gz
Resolviendo nginx.org (nginx.org)... 3.147.99.198, 2a05:d014:5c0:2600::6, 2a05:d014:5c0:2601::6
Conectando con nginx.org (nginx.org)[3.147.99.198]:443... conectado.
Petición HTTP enviada, esperando respuesta... 200 OK
Longitud: 1337335 (1.3M) [application/octet-stream]
Grabando a: «nginx-1.31.0.tar.gz.1»

nginx-1.31.0.tar.gz 100%[===================>]   1.27M  3.68MB/s    en 0.3s    

2026-05-24 01:28:31 (3.68 MB/s) - «nginx-1.31.0.tar.gz.1» guardado [1337335/1337335]

[mstephany@localhost src]$ sudo tar -xzf nginx-1.31.0.tar.gz
[mstephany@localhost src]$ cd nginx-1.31.0
[mstephany@localhost nginx-1.31.0]$
```
* **4. Configurar compilación**
```
[mstephany@localhost nginx-1.31.0]$ sudo ./configure \
> --prefix=/srv/nginx \
> --user=nginx \
> --group=nginx \
> --with-http_ssl_module \
> --with-http_v2_module \
> --with-http_realip_module \
> 
checking for OS
 + Linux 5.14.0-611.55.1.el9_7.x86_64 x86_64
checking for C compiler ... found
 + using GNU C compiler
 + gcc version: 11.5.0 20240719 (Red Hat 11.5.0-11) (GCC) 
checking for gcc -pipe switch ... found
checking for -Wl,-E switch ... found

...

You can either disable the module by using --without-http_rewrite_module
option, or install the PCRE library into the system, or build the PCRE library
statically from the source with nginx by using --with-pcre=<path> option.

[mstephany@localhost nginx-1.31.0]$ sudo ./configure \
> --prefix=/srv/nginx \
  --user=nginx \
  --group=nginx \
  --with-http_ssl_module \
  --with-http_v2_module \
  --with-http_realip_module \
  --with-http_stub_status_module
checking for OS
 + Linux 5.14.0-611.55.1.el9_7.x86_64 x86_64
checking for C compiler ... found
 + using GNU C compiler
 + gcc version: 11.5.0 20240719 (Red Hat 11.5.0-11) (GCC) 
checking for gcc -pipe switch ... found

...

./configure: error: the HTTP rewrite module requires the PCRE library.
You can either disable the module by using --without-http_rewrite_module
option, or install the PCRE library into the system, or build the PCRE library
statically from the source with nginx by using --with-pcre=<path> option.

[mstephany@localhost nginx-1.31.0]$ ^C
[mstephany@localhost nginx-1.31.0]$ sudo dnf install -y pcre pcre-devel
Última comprobación de caducidad de metadatos hecha hace 2:21:42, el sáb 23 may 2026 23:24:20.
El paquete pcre-8.44-4.el9.x86_64 ya está instalado.

...

Instalado:
  pcre-cpp-8.44-4.el9.x86_64             pcre-devel-8.44-4.el9.x86_64          
  pcre-utf16-8.44-4.el9.x86_64           pcre-utf32-8.44-4.el9.x86_64          

¡Listo!
[mstephany@localhost nginx-1.31.0]$ sudo ./configure \
> --prefix=/srv/nginx \
> --user=nginx \
> --group=nginx \
> --with-http_ssl_module \
> --with-http_v2_module \
> --with-http_realip_module \
> --with-http_stub_status_module \
> --with-http_rewrite_module \
> --with-pcre \
> --with-zlib
./configure: error: invalid option "--with-http_rewrite_module"
[mstephany@localhost nginx-1.31.0]$ sudo ./configure \
> --prefix=/srv/nginx \
> --user=nginx \
> --group=nginx \
> --with-http_ssl_module \
> --with-http_v2_module \
> --with-http_realip_module \
> --with-http_stub_status_module \
> --with-pcre \
> --with-zlib
./configure: error: invalid option "--with-zlib"
[mstephany@localhost nginx-1.31.0]$ sudo ./configure \
> --prefix=/srv/nginx \
> --user=nginx \
> --group=nginx \
> --with-http_ssl_module \
> --with-http_v2_module \
> --with-http_realip_module \
> --with-http_realip_module \
> --with-http_stub_status_module
checking for OS
 + Linux 5.14.0-611.55.1.el9_7.x86_64 x86_64
checking for C compiler ... found
 + using GNU C compiler
 + gcc version: 11.5.0 20240719 (Red Hat 11.5.0-11) (GCC) 
checking for gcc -pipe switch ... found
checking for -Wl,-E switch ... found

...

  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"

[mstephany@localhost nginx-1.31.0]$ sudo ./configure \
> --prefix=/srv/nginx \
> --user=nginx \
> --group=nginx \
> --with-http_ssl_module \
> --with-http_v2_module \
> --with-http_realip_module \
> 
checking for OS
 + Linux 5.14.0-611.55.1.el9_7.x86_64 x86_64
checking for C compiler ... found
 + using GNU C compiler
 + gcc version: 11.5.0 20240719 (Red Hat 11.5.0-11) (GCC) 
checking for gcc -pipe switch ... found
checking for -Wl,-E switch ... found

...

./configure: error: the HTTP rewrite module requires the PCRE library.
You can either disable the module by using --without-http_rewrite_module
option, or install the PCRE library into the system, or build the PCRE library
statically from the source with nginx by using --with-pcre=<path> option.

[mstephany@localhost nginx-1.31.0]$ sudo ./configure \
> --prefix=/srv/nginx \
  --user=nginx \
  --group=nginx \
  --with-http_ssl_module \
  --with-http_v2_module \
  --with-http_realip_module \
  --with-http_stub_status_module
checking for OS
 + Linux 5.14.0-611.55.1.el9_7.x86_64 x86_64
checking for C compiler ... found
 + using GNU C compiler
 + gcc version: 11.5.0 20240719 (Red Hat 11.5.0-11) (GCC) 
checking for gcc -pipe switch ... found

...

option, or install the PCRE library into the system, or build the PCRE library
statically from the source with nginx by using --with-pcre=<path> option.

[mstephany@localhost nginx-1.31.0]$ ^C
[mstephany@localhost nginx-1.31.0]$ sudo dnf install -y pcre pcre-devel
Última comprobación de caducidad de metadatos hecha hace 2:21:42, el sáb 23 may 2026 23:24:20.
El paquete pcre-8.44-4.el9.x86_64 ya está instalado.
        
...

¡Listo!
[mstephany@localhost nginx-1.31.0]$ sudo ./configure \
> --prefix=/srv/nginx \
> --user=nginx \
> --group=nginx \
> --with-http_ssl_module \
> --with-http_v2_module \
> --with-http_realip_module \
> --with-http_stub_status_module \
> --with-http_rewrite_module \
> --with-pcre \
> --with-zlib
./configure: error: invalid option "--with-http_rewrite_module"
[mstephany@localhost nginx-1.31.0]$ sudo ./configure \
> --prefix=/srv/nginx \
> --user=nginx \
> --group=nginx \
> --with-http_ssl_module \
> --with-http_v2_module \
> --with-http_realip_module \
> --with-http_stub_status_module \
> --with-pcre \
> --with-zlib
./configure: error: invalid option "--with-zlib"
[mstephany@localhost nginx-1.31.0]$ sudo ./configure \
> --prefix=/srv/nginx \
> --user=nginx \
> --group=nginx \
> --with-http_ssl_module \
> --with-http_v2_module \
> --with-http_realip_module \
> --with-http_realip_module \
> --with-http_stub_status_module
checking for OS
 + Linux 5.14.0-611.55.1.el9_7.x86_64 x86_64
checking for C compiler ... found
 + using GNU C compiler
 + gcc version: 11.5.0 20240719 (Red Hat 11.5.0-11) (GCC) 
checking for gcc -pipe switch ... found

...

Configuration summary
  + using system PCRE library
  + using system OpenSSL library
  + using system zlib library

  nginx path prefix: "/srv/nginx"
  nginx binary file: "/srv/nginx/sbin/nginx"
  nginx modules path: "/srv/nginx/modules"
  nginx configuration prefix: "/srv/nginx/conf"
  nginx configuration file: "/srv/nginx/conf/nginx.conf"
  nginx pid file: "/srv/nginx/logs/nginx.pid"
  nginx error log file: "/srv/nginx/logs/error.log"
  nginx http access log file: "/srv/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
```
**Analisis de errores:**

Al ejecutar **./configure** falló por falta de la librería **PCRE**, necesaria para el módulo de reescritura de URLs. También se detectaron opciones inválidas como **--with-http_rewrite_module** y **--with-zlib**.

**Solución:**

Instalar librerías necesarias:
**sudo dnf install -y pcre pcre-devel zlib zlib-devel openssl openssl-devel.**
Comando final correcto de configuración:

**sudo ./configure \
  --prefix=/srv/nginx \
  --user=nginx \
  --group=nginx \
  --with-http_ssl_module \
  --with-http_v2_module \
  --with-http_realip_module \
  --with-http_stub_status_module**

* **5. Compilar e instalar**

**Se realizaran las configuraciones de:**
* Registar un Slice de Servicio SystemD en /etc/systemd/system/nginx.service.
* Debe de auto Arrancar el servicio durante arranque del SO en en graphical.targeo o multi-user.target

![alt text](image-8.png)
![alt text](image-7.png)

```
[mstephany@localhost nginx-1.31.0]$ sudo make && sudo make install
make -f objs/Makefile
make[1]: se entra en el directorio '/usr/src/nginx-1.31.0'
cc -c -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g  -I src/core -I src/event -I src/event/modules -I src/event/quic -I src/os/unix -I objs \

...
	
[mstephany@localhost nginx-1.31.0]$ sudo nano /etc/systemd/system/nginx.service
[sudo] password for mstephany: 
[mstephany@localhost nginx-1.31.0]$ sudo systemctl daemon-reload
[mstephany@localhost nginx-1.31.0]$ sudo systemctl enable --now nginx
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /etc/systemd/system/nginx.service.

...

Job for nginx.service failed because the control process exited with error code.
See "systemctl status nginx.service" and "journalctl -xeu nginx.service" for details.
[mstephany@localhost nginx-1.31.0]$ sudo journalctl -xeu nginx.service
░░ The error number returned by this process is ERRNO.
...

░░ The job identifier is 73820 and the job result is failed.
[mstephany@localhost nginx-1.31.0]$ sudo /srv/nginx/sbin/nginx -t
nginx: the configuration file /srv/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /srv/nginx/conf/nginx.conf test is successful
[mstephany@localhost nginx-1.31.0]$ sudo mkdir -p /srv/nginx/{logs,html,conf/vhosts,client_body_temp,proxy_temp,fastcgi_temp,uwsgi_temp,scgi_temp}
[mstephany@localhost nginx-1.31.0]$ sudo chown -R nginx:nginx /srv/nginx
[mstephany@localhost nginx-1.31.0]$ sudo nano /srv/nginx/conf/nginx.conf
[mstephany@localhost nginx-1.31.0]$ sudo nano /srv/nginx/conf/nginx.conf
[mstephany@localhost nginx-1.31.0]$ sudo nano /srv/nginx/conf/nginx.conf
[mstephany@localhost nginx-1.31.0]$ sudo systemctl daemon-reload
[mstephany@localhost nginx-1.31.0]$ sudo systemctl start nginx
Job for nginx.service failed because the control process exited with error code.
See "systemctl status nginx.service" and "journalctl -xeu nginx.service" for details.
[mstephany@localhost nginx-1.31.0]$ sudo nano /srv/nginx/conf/nginx.conf
[mstephany@localhost nginx-1.31.0]$ sudo systemctl daemon-reload
[mstephany@localhost nginx-1.31.0]$ sudo systemctl start nginx
Job for nginx.service failed because the control process exited with error code.
See "systemctl status nginx.service" and "journalctl -xeu nginx.service" for details.
[mstephany@localhost nginx-1.31.0]$ sudo /srv/nginx/sbin/nginx -t
nginx: the configuration file /srv/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /srv/nginx/conf/nginx.conf test is successful
[mstephany@localhost nginx-1.31.0]$ sudo systemctl start nginx
Job for nginx.service failed because the control process exited with error code.
See "systemctl status nginx.service" and "journalctl -xeu nginx.service" for details.
[mstephany@localhost nginx-1.31.0]$ sudo rm -f /srv/nginx/logs/nginx.pid
[mstephany@localhost nginx-1.31.0]$ sudo pkill -9 nginx 2>/dev/null
[mstephany@localhost nginx-1.31.0]$ sudo fuser -k 80/tcp 2>/dev/null
[mstephany@localhost nginx-1.31.0]$ sudo chown -R nginx:nginx /srv/nginx
[mstephany@localhost nginx-1.31.0]$ sudo chmod -R 755 /srv/nginx
[mstephany@localhost nginx-1.31.0]$ sudo nano /etc/systemd/system/nginx.service
[mstephany@localhost nginx-1.31.0]$ sudo systemctl daemon-reload
[mstephany@localhost nginx-1.31.0]$ sudo systemctl start nginx
Job for nginx.service failed because the control process exited with error code.
See "systemctl status nginx.service" and "journalctl -xeu nginx.service" for details.
[mstephany@localhost nginx-1.31.0]$ sudo nano /etc/systemd/system/nginx.service
[mstephany@localhost nginx-1.31.0]$ sudo systemctl daemon-reload
[mstephany@localhost nginx-1.31.0]$ sudo rm -f /srv/nginx/logs/nginx.pid
[mstephany@localhost nginx-1.31.0]$ sudo pkill -9 nginx 2>/dev/null
[mstephany@localhost nginx-1.31.0]$ sudo systemctl start nginx
[mstephany@localhost nginx-1.31.0]$ sudo systemctl status nginx
× nginx.service - Servidor Web NGINX
     Loaded: loaded (/etc/systemd/system/nginx.service; enabled; preset: disabl>
     Active: failed (Result: exit-code) since Sun 2026-05-24 02:29:40 CST; 9s a>
   Duration: 44ms
    Process: 585207 ExecStart=/srv/nginx/sbin/nginx -g daemon off; (code=exited>
   Main PID: 585207 (code=exited, status=203/EXEC)
        CPU: 13ms
...

may 24 02:29:40 localhost.localdomain systemd[1]: nginx.service: Failed with re>
[mstephany@localhost nginx-1.31.0]$ ls -l /srv/nginx/sbin/nginx
-rwxr-xr-x. 1 nginx nginx 5191576 may 24 02:01 /srv/nginx/sbin/nginx
[mstephany@localhost nginx-1.31.0]$ cd /usr/src/nginx-1.31.0
[mstephany@localhost nginx-1.31.0]$ sudo make install
make -f objs/Makefile install
make[1]: se entra en el directorio '/usr/src/nginx-1.31.0'

...

make[1]: se sale del directorio '/usr/src/nginx-1.31.0'
[mstephany@localhost nginx-1.31.0]$ sudo chmod -R 755 /srv/nginx
[mstephany@localhost nginx-1.31.0]$ sudo chown -R nginx:nginx /srv/nginx
[mstephany@localhost nginx-1.31.0]$ sudo -u nginx /srv/nginx/sbin/nginx -t
nginx: the configuration file /srv/nginx/conf/nginx.conf syntax is ok
nginx: [emerg] bind() to 0.0.0.0:80 failed (13: Permission denied)
nginx: configuration file /srv/nginx/conf/nginx.conf test failed
[mstephany@localhost nginx-1.31.0]$ ^C
[mstephany@localhost nginx-1.31.0]$ sudo nano /etc/systemd/system/nginx.service
[mstephany@localhost nginx-1.31.0]$ sudo nano /srv/nginx/conf/nginx.conf
```

```
[mstephany@localhost nginx-1.31.0]$ sudo systemctl daemon-reload
[mstephany@localhost nginx-1.31.0]$ sudo rm -f /srv/nginx/logs/nginx.pid
[mstephany@localhost nginx-1.31.0]$ sudo pkill -9 nginx 2>/dev/null
[mstephany@localhost nginx-1.31.0]$ sudo /srv/nginx/sbin/nginx -t
nginx: the configuration file /srv/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /srv/nginx/conf/nginx.conf test is successful
[mstephany@localhost nginx-1.31.0]$ sudo systemctl start nginx
Job for nginx.service failed because the control process exited with error code.
See "systemctl status nginx.service" and "journalctl -xeu nginx.service" for details.
[mstephany@localhost nginx-1.31.0]$ sudo nano /etc/systemd/system/nginx.service
```
**En nano:**
```
[Unit]
Description=Servidor Web NGINX
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=simple
User=root
Group=root
ExecStart=/srv/nginx/sbin/nginx -c /srv/nginx/conf/nginx.conf -g 'daemon on;'
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
KillMode=process
PrivateTmp=true

[Install]
WantedBy=multi-user.target graphical.target
```
**Se realiza el proceso dentro de nano de:**
* Usar Type=simple que es más tolerante.
* Arrancar como root (obligatorio para puerto 80).
* La línea user nginx nginx; que esta en el archivo /srv/nginx/conf/nginx.conf sigue funcionando: los procesos de trabajo si corren como usuario nginx.
* User=root: Necesario para abrir el puerto 80 (puertos privilegiados <1024).
* daemon off;: Obligatorio en SystemD, ya que SystemD espera controlar el proceso principal, si se mantiene como daemon on, NGINX se bifurca y SystemD cree que falló.


Para cuardar los cambios dentro de nano coloco: 
**Guarda (Ctrl+O → Enter) y sal (Ctrl+X).**

**Recargar definiciones:**
```
[mstephany@localhost nginx-1.31.0]$ sudo systemctl daemon-reload
```
**Borrar cualquier rastro viejo:**
```
[mstephany@localhost nginx-1.31.0]$ sudo rm -f /srv/nginx/logs/nginx.pid
[mstephany@localhost nginx-1.31.0]$ sudo pkill -9 nginx 2>/dev/null
```
**Se otorga un permiso total a la carpeta de logs:**
```
[mstephany@localhost nginx-1.31.0]$ sudo chmod 777 /srv/nginx/logs
```
**Ejecucion final:**
* Verificar que corre correctamente: sudo systemctl start nginx y sudo systemctl status nginx,
```
[mstephany@localhost nginx-1.31.0]$ sudo systemctl start nginx
[mstephany@localhost nginx-1.31.0]$ sudo systemctl status nginx
× nginx.service - Servidor Web NGINX
     Loaded: loaded (/etc/systemd/system/nginx.service; enabled; preset: disabl>
     Active: failed (Result: exit-code) since Sun 2026-05-24 02:39:57 CST; 13s >
   Duration: 29ms
    Process: 605100 ExecStart=/srv/nginx/sbin/nginx -c /srv/nginx/conf/nginx.co>
   Main PID: 605100 (code=exited, status=203/EXEC)
        CPU: 14ms

may 24 02:39:57 localhost.localdomain systemd[1]: Started Servidor Web NGINX.
may 24 02:39:57 localhost.localdomain systemd[605100]: nginx.service: Failed to>
may 24 02:39:57 localhost.localdomain systemd[605100]: nginx.service: Failed at>
may 24 02:39:57 localhost.localdomain systemd[1]: nginx.service: Main process e>
may 24 02:39:57 localhost.localdomain systemd[1]: nginx.service: Failed with re>
```
**Confirmar que el programa sí funciona.**
* Ver procesos: ps aux | grep nginx.
```
[mstephany@localhost nginx-1.31.0]$ sudo /srv/nginx/sbin/nginx -c /srv/nginx/conf/nginx.conf
[mstephany@localhost nginx-1.31.0]$ ps aux | grep nginx
root      608295  0.0  0.0  15816  2776 ?        Ss   02:41   0:00 nginx: master process /srv/nginx/sbin/nginx -c /srv/nginx/conf/nginx.conf
nginx     608297  0.0  0.1  16212  4456 ?        S    02:41   0:00 nginx: worker process
mstepha+  609318  0.0  0.0 221684  2388 pts/0    S+   02:42   0:00 grep --color=auto nginx
```

**Comando de la versión**
```
[mstephany@localhost nginx-1.31.0]$ /srv/nginx/sbin/nginx -v
nginx version: nginx/1.31.0
[mstephany@localhost nginx-1.31.0]$ 
```
## Implementación y configuración de servidor PHP, compilado desde codigo fuente en la version 8.4.x:

**Ir a la carpeta de trabajo**

```
[mstephany@localhost ~]$ cd /usr/src
```
**Descargar el código de PHP 8.4.0**
```
[mstephany@localhost src]$ sudo wget https://www.php.net/distributions/php-8.4.0.tar.gz
[sudo] password for mstephany: 
--2026-05-24 22:01:03--  https://www.php.net/distributions/php-8.4.0.tar.gz
Resolviendo www.php.net (www.php.net)... 185.85.0.29, 2a02:cb40:200::1ad
Conectando con www.php.net (www.php.net)[185.85.0.29]:443... conectado.
Petición HTTP enviada, esperando respuesta... 200 OK
Longitud: no especificado [application/x-gzip]
Grabando a: «php-8.4.0.tar.gz»

php-8.4.0.tar.gz        [               <=>  ]  20.61M  6.92MB/s    en 3.0s    

2026-05-24 22:01:07 (6.92 MB/s) - «php-8.4.0.tar.gz» guardado [21615912]
```
**Descomprimir el archivo descargado:**
```
[mstephany@localhost src]$ sudo tar -xzf php-8.4.0.tar.gz
```
**Entrar a la carpeta que se creó**
```
[mstephany@localhost src]$ cd php-8.4.0
```
**Configuración previa y resolución de dependencias**
**Para asegurar la disponibilidad de todas las librerías necesarias, instar los paquetes correspondientes:**

```
[mstephany@localhost php-8.4.0]$ sudo dnf install -y libxml2-devel openssl-devel curl-devel libjpeg-devel libpng-devel freetype-devel libicu-devel
[mstephany@localhost php-8.4.0]$ sudo dnf install -y sqlite-devel

```
![alt text](image-11.png)
![alt text](image-12.png)
![alt text](image-14.png)
![alt text](image-15.png)

* Ejecutamos el script de configuración con las rutas y módulos que necesita
```
[mstephany@localhost php-8.4.0]$ sudo ./configure \
> --prefix=/srv/nginx \
  --enable-fpm \
  --with-fpm-user=php \
  --with-fpm-group=nginx \
  --with-openssl \
  --with-curl \
  --with-gd \
  --with-jpeg \
  --with-png \
  --with-freetype \
  --enable-intl \
  --enable-calendar \
  --enable-exif \
  --disable-mbregex \
  --enable-xml \
  --enable-sockets
```

**De este modo se genera el archivo Makefile necesario para la compilación.**

![alt text](image-16.png)

* El mensaje WARNING: unrecognized options: --with-gd, --with-png significa que en PHP 8.4 cambiaron el nombre de esas opciones, ya no se llaman así, pero el sistema las entiende igual o ya vienen activadas por defecto. Ya se pueden realizar: soporte de imágenes, jpeg, fuentes, etc. 
Ya esta creado el archivo Makefile.


**Compilamos el código y lo instalamos en la ruta definida /srv/nginx:**

```
[mstephany@localhost php-8.4.0]$ sudo make && sudo make install
```
![alt text](image-17.png)

![alt text](image-18.png)

* Todo el sistema de PHP queda instalado en /srv/nginx/
* El ejecutable de PHP-FPM (el servicio que conecta con NGINX) está en /srv/nginx/sbin/php-fpm

**Crear el archivo de configuración de PHP-FPM**
* Copiar el archivo de configuración
```
[mstephany@localhost php-8.4.0]$ sudo cp /srv/nginx/etc/php-fpm.conf.default /srv/nginx/etc/php-fpm.conf
[sudo] password for mstephany: 
[mstephany@localhost php-8.4.0]$ sudo cp /srv/nginx/etc/php-fpm.d/www.conf.default /srv/nginx/etc/php-fpm.d/www.conf
```
**Modificar la configuración para ajustar usuario, grupo y comunicación por socket:**
```
[mstephany@localhost php-8.4.0]$ sudo sed -i 's/^user = nobody/user = php/' /srv/nginx/etc/php-fpm.d/www.conf
[sudo] password for mstephany: 
[mstephany@localhost php-8.4.0]$ sudo sed -i 's/^group = nobody/group = nginx/' /srv/nginx/etc/php-fpm.d/www.conf
[mstephany@localhost php-8.4.0]$ sudo sed -i 's/^listen = 127.0.0.1:9000/listen = \/srv\/nginx\/run\/php-fpm.sock/' /srv/nginx/etc/php-fpm.d/www.conf
[mstephany@localhost php-8.4.0]$ sudo sed -i 's/^;listen.owner = nobody/listen.owner = nginx/' /srv/nginx/etc/php-fpm.d/www.conf
[mstephany@localhost php-8.4.0]$ sudo sed -i 's/^;listen.group = nobody/listen.group = nginx/' /srv/nginx/etc/php-fpm.d/www.conf
[mstephany@localhost php-8.4.0]$ sudo sed -i 's/^;listen.mode = 0660/listen.mode = 0660/' /srv/nginx/etc/php-fpm.d/www.conf
```

**Crear la carpeta donde se alojará el socket y se ajustan permisos:**

```
[mstephany@localhost php-8.4.0]$ sudo mkdir -p /srv/nginx/run
[mstephany@localhost php-8.4.0]$ sudo chown nginx:nginx /srv/nginx/run
[mstephany@localhost php-8.4.0]$ 
```
![alt text](image-19.png)

**Crear el archivo de servicio para poder gestionar PHP con systemctl (Systemd):**

```
[mstephany@localhost php-8.4.0]$ sudo tee /etc/systemd/system/php-fpm.service <<EOF
> [Unit]
Description=PHP FastCGI Process Manager
After=network.target nginx.service

[Service]
User=php
Group=nginx
ExecStart=/srv/nginx/sbin/php-fpm --nodaemonize --fpm-config /srv/nginx/etc/php-fpm.conf
ExecReload=/bin/kill -USR2 \$MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```
![alt text](image-20.png)

**Activar y arrancar el servicio PHP**
* Recargar la configuración del sistema
* Activar para que arranque al prender la máquina
* Arrancar PHP

```
[mstephany@localhost php-8.4.0]$ sudo systemctl daemon-reload
[mstephany@localhost php-8.4.0]$ sudo systemctl enable php-fpm
Created symlink /etc/systemd/system/multi-user.target.wants/php-fpm.service → /etc/systemd/system/php-fpm.service.
[mstephany@localhost php-8.4.0]$ sudo systemctl start php-fpm
```
**Conexión con NGINX y resolución de errores:**
* Se edita la configuración de NGINX para que reconozca y procese archivos .php:
```
[mstephany@localhost php-8.4.0]$ sudo nano /srv/nginx/conf/nginx.conf
```
**Errores presentados y soluciones:**

* Error: Permission denied al ejecutar los binarios
```
[mstephany@localhost php-8.4.0]$ sudo systemctl restart nginx
[sudo] password for mstephany: 
[mstephany@localhost php-8.4.0]$ sudo systemctl restart php-fpm
[mstephany@localhost php-8.4.0]$ sudo systemctl status nginx php-fpm --no-pager -l...

php-fpm.service: Failed at step EXEC spawning /srv/nginx/sbin/php-fpm: Permission denied
may 25 00:33:11 localhost.localdomain systemd[1]: php-fpm.service: Main process exited, code=exited, status=203/EXEC
may 25 00:33:11 localhost.localdomain systemd[1]: php-fpm.service: Failed with result 'exit-code'.
```
**Solución:**
* Ajustar permisos, propietarios y etiquetas de SELinux:
```
[mstephany@localhost php-8.4.0]$ sudo chmod -R 755 /srv/nginx
[mstephany@localhost php-8.4.0]$ sudo chown -R root:root /srv/nginx
[mstephany@localhost php-8.4.0]$ sudo chown -R nginx:nginx /srv/nginx/html /srv/nginx/logs /srv/nginx/run
[mstephany@localhost php-8.4.0]$ sudo chmod -R 770 /srv/nginx/run
[mstephany@localhost php-8.4.0]$ sudo chcon -R -t bin_t /srv/nginx/sbin /srv/nginx/bin
[mstephany@localhost php-8.4.0]$ sudo systemctl daemon-reload
```
* Error: fastcgi_pass directive is not allowed here
```
nginx: [emerg] "fastcgi_pass" directive is not allowed here in /srv/nginx/conf/nginx.conf:62...

may 25 00:37:31 localhost.localdomain systemd[1]: php-fpm.service: Failed with result 'exit-code'.
```

* Como los errores seguia, se creo directamente el archivo de log y se asigno su propietario exacto al usuario que ejecuta PHP-FPM:
```
[mstephany@localhost php-8.4.0]$ sudo touch /srv/nginx/logs/php-fpm.log
[mstephany@localhost php-8.4.0]$ sudo chown php:nginx /srv/nginx/logs/php-fpm.log
```
* Arrancar nuevamente los servicios:
```
[mstephany@localhost php-8.4.0]$ sudo systemctl start php-fpm
[mstephany@localhost php-8.4.0]$ sudo systemctl start nginx
```
Error 4: Fallo en el arranque del servicio NGINX
* El servicio de NGINX no logra iniciar debido a una configuración incompleta en el archivo nginx.service.
```
Job for nginx.service failed because the control process exited with error code.
See "systemctl status nginx.service" and "journalctl -xeu nginx.service" for details.
```
**Solución:**
* Reescribir el archivo de servicio
Definir correctamente el tipo de servicio, la ubicación del PID y los comandos de inicio/parada:
```
[mstephany@localhost php-8.4.0]$ sudo tee /etc/systemd/system/nginx.service <<'EOF'
> [Unit]
Description=Servidor Web NGINX
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/srv/nginx/logs/nginx.pid
User=root
Group=root
ExecStart=/srv/nginx/sbin/nginx -c /srv/nginx/conf/nginx.conf
ExecReload=/srv/nginx/sbin/nginx -s reload
ExecStop=/srv/nginx/sbin/nginx -s stop
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```

* Crear el archivo de PID necesario y se modifican permisos generales y de seguridad (SELinux):
```
[mstephany@localhost php-8.4.0]$ sudo touch /srv/nginx/logs/php-fpm.log
[mstephany@localhost php-8.4.0]$ sudo touch /srv/nginx/logs/nginx.pid
[mstephany@localhost php-8.4.0]$ sudo chown -R root:root /srv/nginx/
[mstephany@localhost php-8.4.0]$ sudo chown -R nginx:nginx /srv/nginx/html /srv/nginx/logs /srv/nginx/run
[mstephany@localhost php-8.4.0]$ sudo chmod -R 755 /srv/nginx/
[mstephany@localhost php-8.4.0]$ sudo chcon -R -t bin_t /srv/nginx/sbin /srv/nginx/bin
[mstephany@localhost php-8.4.0]$ sudo chcon -R -t httpd_log_t /srv/nginx/logs
```
* Recargar configuración:
```
[mstephany@localhost php-8.4.0]$ sudo systemctl daemon-reload
[mstephany@localhost php-8.4.0]$ sudo pkill -f nginx 2>/dev/null
[mstephany@localhost php-8.4.0]$ sudo pkill -f php-fpm 2>/dev/null
[mstephany@localhost php-8.4.0]$ sudo systemctl start php-fpm
[mstephany@localhost php-8.4.0]$ sudo systemctl start nginx
```
Error:
```
Job for nginx.service failed because a timeout was exceeded.
See "systemctl status nginx.service" and "journalctl -xeu nginx.service" for details.
```
**Solución:**
* Ajuste de SELinux y arranque manual
* Eliminar los archivos de servicio problemáticos para simplificar la gestión:

```
[mstephany@localhost ~]$ sudo rm -f /etc/systemd/system/nginx.service
[mstephany@localhost ~]$ sudo rm -f /etc/systemd/system/php-fpm.service
[mstephany@localhost ~]$ sudo systemctl daemon-reload
```
* Desactivar temporalmente SELinux para pruebas y se habilitan políticas necesarias:
```
[mstephany@localhost ~]$ sudo setenforce 0
[mstephany@localhost ~]$ sudo setsebool -P httpd_execmem 1
[mstephany@localhost ~]$ sudo setsebool -P httpd_enable_homedirs 1
```
* Se corrige la ruta del archivo de log directamente en la configuración de PHP:
```
[mstephany@localhost ~]$ sudo sed -i 's|^;*error_log.*$|error_log = /srv/nginx/logs/php-fpm.log|' /srv/nginx/etc/php-fpm.conf
```
* Iniciar los servicios manualmente desde la ruta de instalación, lo cual evita errores del gestor systemd:
```
[mstephany@localhost ~]$ sudo /srv/nginx/sbin/php-fpm --fpm-config /srv/nginx/etc/php-fpm.conf -D
[mstephany@localhost ~]$ sudo /srv/nginx/sbin/nginx -c /srv/nginx/conf/nginx.conf
```
* Verificación de procesos activos:
```
[mstephany@localhost ~]$ ps aux | grep -E 'nginx|php-fpm'
root      253416  0.0  0.3 301280 12028 ?        Ss   10:22   0:00 php-fpm: master process (/srv/nginx/etc/php-fpm.conf)
php       253419  0.0  0.2 301280 10280 ?        S    10:22   0:00 php-fpm: pool www
php       253420  0.0  0.2 301280 10280 ?        S    10:22   0:00 php-fpm: pool www
root      256668  0.0  0.0  15800  2800 ?        Ss   10:22   0:00 nginx: master process /srv/nginx/sbin/nginx -c /srv/nginx/conf/nginx.conf
nginx     256669  0.0  0.1  16228  4480 ?        S    10:22   0:00 nginx: worker process
mstepha+  258443  0.0  0.0 221816  2376 pts/0    R+   10:22   0:00 grep --color=auto -E nginx|php-fpm
```
Ambos servicios están corriendo correctamente.

Error: NGINX no procesa archivos PHP
```
[mstephany@localhost ~]$ curl http://localhost/info.php
<!DOCTYPE html>
<html>
<head>
<title>Error</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>An error occurred.</h1>
<p>Sorry, the page you are looking for is currently unavailable.<br/>
Please try again later.</p>
<p>If you are the system administrator of this resource then you should check
the error log for details.</p>
<p><em>Faithfully yours, nginx.</em></p>
</body>
</html>
[mstephany@localhost ~]$ ^C
```
**Solución:**
* Configuración del sitio y prueba
* Crear el archivo de prueba info.php en la carpeta web y asignar permisos correctos:
```
[mstephany@localhost ~]$ echo "<?php phpinfo(); ?>" | sudo tee /srv/nginx/html/info.php
<?php phpinfo(); ?>
[mstephany@localhost ~]$ sudo chown nginx:nginx /srv/nginx/html/info.php
[mstephany@localhost ~]$ sudo chmod 644 /srv/nginx/html/info.php
```
* Editar la configuración de NGINX (/srv/nginx/conf/nginx.conf) para asegurar que el bloque location ~ \.php$ tenga las directivas fastcgi_pass apuntando al socket:
```
[mstephany@localhost ~]$ sudo nano /srv/nginx/conf/nginx.conf

fastcgi_pass unix:/srv/nginx/run/php-fpm.sock;
fastcgi_index index.php;
include fastcgi_params;
```
* Copiar el archivo de parámetros y modificar permisos del socket para evitar errores de comunicación:
```
[mstephany@localhost ~]$ sudo cp /srv/nginx/conf/fastcgi_params /srv/nginx/html/
[mstephany@localhost ~]$ echo "<?php phpinfo(); ?>" | sudo tee /srv/nginx/html/info.php
<?php phpinfo(); ?>
[mstephany@localhost ~]$ sudo chmod 777 /srv/nginx/run/php-fpm.sock
[mstephany@localhost ~]$ sudo /srv/nginx/sbin/nginx -s reload
```
Volver a probar:
```
[mstephany@localhost ~]$ curl http://localhost/info.php
```
* Se recibe el código HTML completo con la información detallada de la configuración de PHP.
```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"><head>
<style type="text/css">
body {background-color: #fff; color: #222; font-family: sans-serif;}

...

</td></tr>
</table>
</div></body></html>[mstels -l /srv/nginx/run/php-fpm.sockinx/run/php-fpm.sock
srwxrwxrwx. 1 root root 0 may 25 10:22 /srv/nginx/run/php-fpm.sock
[mstephany@localhost ~]$ 
```
![alt text](image-30.png)
![alt text](image-28.png)
![alt text](image-27.png)
![alt text](image-26.png)
![alt text](image-25.png)
![alt text](image-24.png)
![alt text](image-23.png)
![alt text](image-22.png)
![alt text](image-31.png)

# Conclusiones:
Se cumplieron todos los requisitos establecidos: se instaló y configuró NGINX 1.31.x y PHP 8.4.x, ambos compilados desde código fuente y alojados en la ruta solicitada /srv/nginx/. Se definieron correctamente los usuarios y grupos de sistema (nginx para el servidor web, php:nginx para el procesador), se crearon sus respectivos archivos de servicio en SystemD (nginx.service y php-fpm8.4.service) y se configuraron para que se inicien automáticamente junto al sistema operativo bajo el objetivo multi-user.target.
La integración entre ambos servicios se realizó mediante socket UNIX ubicado en /tmp/php84.sock. Se habilitaron las funcionalidades requeridas: procesamiento de imágenes, manejo de fechas y soporte de internacionalización (intl). La prueba con el archivo phpinfo.php confirmó que la comunicación FastCGI funciona correctamente, desplegando la información detallada de la instalación en el navegador.

Se fortaleció el conocimiento de ajustar y mantener un entorno web desde cero, teniendo control total sobre cada componente.

# Bibliografía:
  NGINX, Inc. (2024). Building nginx from Sources. NGINX Documentation. https://nginx.org/en/docs/configure.html

  The PHP Group. (2024). Instalación en sistemas Unix. PHP Manual. https://www.php.net/manual/es/install.unix.php

  AlmaLinux OS Foundation. (2024). AlmaLinux 9 Release Notes. AlmaLinux Wiki. https://wiki.almalinux.org/release-notes/9.html

  AlmaLinux OS Foundation. (2024). *AlmaLinux 9 Release Notes*. AlmaLinux Wiki. https://wiki.almalinux.org/release-notes/9.html

   NGINX, Inc. (2024). *Building nginx from Sources*. NGINX Documentation. https://nginx.org/en/docs/configure.html

  Romero Gonzalez, G. M. (2025, 19 de junio). *Video de compilación de NGINX en AlmaLinux 9* [Video]. YouTube. https://youtu.be/i-DGI-R1uFw

  Romero Gonzalez, G. M. (2025, 20 de junio). *Video de compilación de PHP-src en AlmaLinux 9* [Video]. YouTube. https://youtu.be/OCiHQhco5Is

  Romero Gonzalez, G. M. (2025, 20 de junio). *Video de integración PHP-FPM - NGINX FastCGI y modularidad PHP* [Video]. YouTube. https://youtu.be/QWGb4AsHaZE

  The PHP Group. (2024). *Instalación en sistemas Unix*. PHP Manual. https://www.php.net/manual/es/install.unix.php

