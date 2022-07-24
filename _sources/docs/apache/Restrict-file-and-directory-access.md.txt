# Restrict file and directory access

To restrict a directory from access by users, deny all users using the `Directory` directive:

```
<Directory "/var/www/directory">
    Order Deny,Allow
    Deny from all
    Allow from 192.168.1.0/24
    Allow from .core.com
</Directory>
```

To restrict a file using the `File` directive:

```
# The following lines prevent .htaccess and .htpasswd files from being 
# viewed by Web clients. 
#
<Files ~ "^\.ht">
    Order allow,deny
    Deny from all
    Satisfy all
</Files>
```

To limit the scope of enclosed directives by URL use the `Location` directive:

```
<Location /admin>
    Order Deny,Allow
    Deny from all
    Allow from 192.168.1.0/24
    Allow from .core.com
</Location>
```

