# Disable SSI and CGI execution

## CGI 

Nginx cannot directly execute external programs (CGI). Phew. But if you really really want it to, use [fcgi](https://www.nginx.com/resources/wiki/start/topics/examples/fastcgiexample/) securely.

## Disable SSI and autoindex execution 

The `ngx_http_autoindex_module` processes requests ending with the slash character `/` and produces a directory listing. Usually a request is passed to the `ngx_http_autoindex_module` when the `ngx_http_index_module` cannot find an index file.

You can configure and install Nginx using only required modules. To see which modules can be turned on or off while compiling the Nginx server:

    # ./configure --help | less

To disable the SSI and autoindex module:

    # ./configure --without-http_autoindex_module --without-http_ssi_module
    # make
    # make install


