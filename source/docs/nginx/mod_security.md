# Install and use mod_security

ModSecurity was originally developed for Apache, but is now also available for Nginx (and other platforms) in a "standalone" version.

To install it without (re)compiling Nginx, install the dependencies:

    # apt-get install libxml2 libxml2-dev libxml2-utils libaprutil1 libaprutil1-dev

And download, compile and install `mod_security`:

    # git clone https://github.com/SpiderLabs/ModSecurity.git mod_security
    # cd mod_security
    ~/mod_security# ./autogen.sh
    ~/mod_security# ./configure --enable-standalone-module
    ~/mod_security# make

To compile Nginx from source with the modsecurity module (check for latest version here: http://nginx.org/en/download.html)

    # wget http://www.nginx.org/download/nginx-1.9.5.tar.gz
    # tar -xvpzf nginx-1.9.5.tar.gz
    # cd nginx-1.9.5
    ~/nginx-1.9.5# ./configure --add-module=../mod_security/nginx/modsecurity
    ~/nginx-1.9.5# make
    ~/nginx-1.9.5# make install

The ModSecurity configuration file must be defined in the `nginx.conf` file, something like this:

    server {
       listen       80;
       server_name  localhost;

       location / {
          ModSecurityEnabled on;
          ModSecurityConfig modsecurity.conf;
       }

    }

For custom rules applied to different directories, create new mod_security.conf files, for example:

    location /secured {
       ModSecurityConfig modsecurity3.conf; 
       proxy_pass http://secured.core.com/;
       proxy_read_timeout 180s;
    }

To turn it off for a particular directory:

    location /unsecured/ {
       ModSecurityEnabled off;
       proxy_pass http://unsecured.core.com/;
       proxy_read_timeout 180s;
    }

Restart Nginx. 

Have a look at the [OWASP ModSecurity Core Rule Set Project](https://www.owasp.org/index.php/Category:OWASP_ModSecurity_Core_Rule_Set_Project) (CRS) for making more secure changes. The CRS aims to protect web applications from a wide range of attacks, with a minimum of false alerts.
