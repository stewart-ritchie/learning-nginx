# nginx-ref

Learning material for NGINX.

## Running in Docker

An easy approach to running NGINX is to pull and run the Docker image. We start by pulling the latest image and then run NGINX in detached server mode. The local 8080 port is mapped to the default port on which NGINX listens (80).

```
docker pull nginx
docker run -p 8080:80 -d nginx
```

You can connect to the container using the container id returned by _docker run_, or at any time by listing your docker containers.

```
docker container ls -a
```

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                  PORTS                  NAMES
42e6ba0fde39        nginx               "nginx -g 'daemon of…"   5 minutes ago       Up 5 minutes            0.0.0.0:8080->80/tcp   eloquent_hellman
```

```
docker exec -it 42e6ba0fde39 bash
```

You should now be connected to the NGINX instance with bash. By default you get root access with Docker (no _sudo_ needed).

```
root@42e6ba0fde39:/#
```

## Installing a file editor

Docker images are very bare, so we need to install a file editor. Start by updating the package list. Then, install an editor. _Nano_ is a good choice for beginners but _vim_ tends to be used more commonly. 

```
apt-get update
apt-get install nano
apt-get install vim
```

Note: if you need to familiarise yourself with the editor, do so now (we won't cover those details here).

## NGINX configuration files

Configuration of NGINX is performed by editing the _.conf_ files in the NGINX root directory and sub-directories.

```
cd etc/nginx/
ls -l
```

```
drwxr-xr-x 2 root root 4096 Aug 15 21:22 conf.d
-rw-r--r-- 1 root root 1007 Aug 13 08:50 fastcgi_params
-rw-r--r-- 1 root root 2837 Aug 13 08:50 koi-utf
-rw-r--r-- 1 root root 2223 Aug 13 08:50 koi-win
-rw-r--r-- 1 root root 5231 Aug 13 08:50 mime.types
lrwxrwxrwx 1 root root   22 Aug 13 08:50 modules -> /usr/lib/nginx/modules
-rw-r--r-- 1 root root  643 Aug 13 08:50 nginx.conf
-rw-r--r-- 1 root root  636 Aug 13 08:50 scgi_params
-rw-r--r-- 1 root root  664 Aug 13 08:50 uwsgi_params
-rw-r--r-- 1 root root 3610 Aug 13 08:50 win-utf
```

Let's take a look at the root _nginx.conf_ configuration file. The file is structured using _directives_ some of which are scoped into _blocks_. The root configuration file tells us about the locations of other interesting files, e.g. /var/log/nginx/error.log. The _include_ directives are used to inline other configuration files so we have reasonable way of structuring complex configurations and re-using configuration snippets.

```Nginx
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```
