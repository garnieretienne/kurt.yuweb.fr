title: build_nginx_with_upload_progress_module

Nginx Upload Progress Module
============================

> nginx_uploadprogress_module is an implementation of an upload progress system, that monitors RFC1867 POST upload as they are transmitted to upstream servers.
> It works by tracking the uploads proxied by Nginx to upstream servers without analysing the uploaded content and offers a web API to report upload progress in Javascript, JSON or configurable format. It works because Nginx acts as an accelerator of an upstream server, storing uploaded POST content on disk, before transmitting it to the upstream server. Each individual POST upload request should contain a progress unique identifier.

Source: http://wiki.nginx.org/HttpUploadProgressModule

Installation
============

Step 1: download and extract Nginx
----------------------------------

```bash
wget http://nginx.org/download/nginx-1.0.4.tar.gz
tar xzvf nginx-1.0.4.tar.gz
```

Step 2: download and extract Upload Progress Module
---------------------------------------------------

```bash
wget https://github.com/masterzen/nginx-upload-progress-module/tarball/v0.8.2
mv v0.8.2 nginx-upload-progress-module-0.8.2.tar.gz
tar xzvf nginx-upload-progress-module-0.8.2.tar.gz
mv masterzen-nginx-upload-progress-module-* nginx-upload-progress-module-0.8.2
```
Step 3: configure and compile Nginx with Upload progess module
--------------------------------------------------------------

```bash
cd nginx-1.0.4
./configure --add-module=../nginx-upload-progress-module-0.8.2
make
make install
```

Error: `error: variable ‘c’ set but not used`
--------------------------------------------

I use this configuration:

- gcc: v4.6.0
- nginx: v1.0.4
- upload_progress_module: v0.8.2

And i encoured this error:

```
gcc -c -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Wunused-function -Wunused-variable -Wunused-value -Werror -g   -I src/core -I src/event -I src/event/modules -I src/os/unix -I objs -I src/http -I src/http/modules -I src/mail \
	-o objs/addon/nginx-upload-progress-module-0.8.2/ngx_http_uploadprogress_module.o \
	../nginx-upload-progress-module-0.8.2/ngx_http_uploadprogress_module.c
../nginx-upload-progress-module-0.8.2/ngx_http_uploadprogress_module.c: In function ‘ngx_http_uploadprogress_event_handler’:
../nginx-upload-progress-module-0.8.2/ngx_http_uploadprogress_module.c:452:50: error: variable ‘c’ set but not used [-Werror=unused-but-set-variable]
../nginx-upload-progress-module-0.8.2/ngx_http_uploadprogress_module.c: In function ‘ngx_http_uploadprogress_cleanup’:
../nginx-upload-progress-module-0.8.2/ngx_http_uploadprogress_module.c:1010:38: error: variable ‘ctx’ set but not used [-Werror=unused-but-set-variable]
../nginx-upload-progress-module-0.8.2/ngx_http_uploadprogress_module.c: In function ‘ngx_http_uploadprogress_create_loc_conf’:
../nginx-upload-progress-module-0.8.2/ngx_http_uploadprogress_module.c:1238:43: error: variable ‘t’ set but not used [-Werror=unused-but-set-variable]
../nginx-upload-progress-module-0.8.2/ngx_http_uploadprogress_module.c: In function ‘ngx_http_uploadprogress_init_variables_and_templates’:
../nginx-upload-progress-module-0.8.2/ngx_http_uploadprogress_module.c:1312:43: error: variable ‘n’ set but not used [-Werror=unused-but-set-variable]
../nginx-upload-progress-module-0.8.2/ngx_http_uploadprogress_module.c:1310:43: error: variable ‘t’ set but not used [-Werror=unused-but-set-variable]
cc1: all warnings being treated as errors

make[1]: *** [objs/addon/nginx-upload-progress-module-0.8.2/ngx_http_uploadprogress_module.o] Error 1
make[1]: Leaving directory `/home/kurt/build/nginx-1.0.4'
make: *** [build] Error 2

```

Theses warnings are treated as errors by gcc and stop the compilation. This is due to the `-Werror` argument in the gcc command.
To fix this problem, simply edit `nginx-1.0.4/objs/Makefile` and delete this argument from the gcc command (`CFLAGS`).

```bash
make && make install
```
