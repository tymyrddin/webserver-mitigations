# Hide version and OS identity

Edit `src/http/ngx_http_header_filter_module.c`:

    # vi +48 src/http/ngx_http_header_filter_module.c

Find:

    static char ngx_http_server_string[] = "Server: nginx" CRLF;
    static char ngx_http_server_full_string[] = "Server: " NGINX_VER CRLF;

Change to:

    static char ngx_http_server_string[] = "Server: You wish" CRLF;
    static char ngx_http_server_full_string[] = "Server: You wish" CRLF;

Save and close. Compile server.

For turning off Nginx version number displayed on all auto generated error pages add the following line to `/usr/local/nginx/conf/nginx.conf`:

    server_tokens off

