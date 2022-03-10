Created for handling 10k of concurrent connections with the focus on
- High Performance
- High Concurrency
- Low Resource Usage

***Official Definition***
NGINX is open source software for web serving, reverse proxying, caching, load balancing, media streaming, and more. It started out as a web server designed for maximum performance and stability. In addition to its HTTP server capabilities, NGINX can also function as a proxy server for email (IMAP, POP3, and SMTP) and a reverse proxy and load balancer for HTTP, TCP, and UDP servers.

## Nginx vs Apache
The main difference between Apache and NGINX lies in their design architecture. Apache uses a process-driven approach and creates a new thread for each request. Whereas NGINX uses an event-driven architecture to handle multiple requests within one thread.

<p>

***Nginx Features***
- Event-driven
- Asynchronous
- Non-blocking

***Result***
- Faster Static Resources
- High Concurrency

## Configuration
Two main configuration terms are context and directive.
- Directive is a spesific configuration includes a name and a value
- Context is section within the configuration. Its like a scope. Context are nested and inherited from the parent.

### Creating a Virtual Host
It's gonna serve static files from a directory on your server.
Let's say that we have an html, css and a png file.

- nginx.conf file is located /etc/nginx.

The default conf looks like this

```

user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
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

In order to define a virtual host we need at least these configs
```
events {}

http {

}
```

- First, we define server context or server block.
- It essentially will listen on port 80
- Next we set the `server_name` which is the domain, subdomain or an ip address for which the server context exists.
- Finally define a root directive. This will be the root path from which nginx will be serving requests or interpreting static requests from.

```
events {}

http {

  server {

    listen 80;
    server_name 167.99.93.26;

    root /sites/demo;
  }
}
```

When we first reload this configs we can't see the css in browsers. When we check the browser the style file loaded.
<br>
So, what is the problem?
<p>
Nginx is sending the wrong mime type with the stylesheet. We can confirm this by requesting the style.css.
<br>
When we write `curl -I http://167.99.93.26/style.css`, we can see that Content-Type: text/plain.
<br>
To fix that we can provide content type in the conf file. But there is another way to do that. mime.types are located in etc/nginx folder. We simply include it.

```
events {}

http {

  include mime.types;

  server {

    listen 80;
    server_name 167.99.93.26;

    root /sites/demo;
  }
}
```
 
## Location Blocks
The most used context in any nginx configuration. Think of location as intercepting a request based on its value and then
doing something other than just trying to serve a matching file relative to the root directory as we saw in the previous.
<br>
The standard behavior is great for static resources. But if we request something like domain/greet, domain/users and etc.,
nginx can not match a file and instead serves default 404 page.

<p>
Location context takes paremeter uri.
```
location /greet{
      return 200 'Hello from NGINX "/greet" location.';
    }
```

There are several ways to match uri in location blocks.

- Prefix match => anything started with /greet will match. For ex /greetings, /greet/user
- Exact match => `location = /greet`
- Regex match => `location ~/greet[0-9]`, /greet will not match. /greet1 will match. Note that ~ is case sensitive.
- Regex case insensitive => `location ~* /greet[0-9]`
- Preferential prefix match => `location ^~ /greet`. Same as prefix but more important than regex.

There is priority in these matches. For ex: regex is prior from prefix.
The order is like that : `exact -> preferential -> regex -> prefix`

## Variables
Nginx has variables and we can also define custom variables as well.
<br>

```
set $mon 'No';

    # Check if weekend
    if ( $date_local ~ 'Monday' ) {
      set $mon 'Yes';
    }
```


We can also use statements. 

```
if ($request_method = POST ) {
  return 405;
}
```
It is generally considered as a good idea to avoid using if. https://www.nginx.com/resources/wiki/start/topics/depth/ifisevil/

## Rewrites and Redirects
???