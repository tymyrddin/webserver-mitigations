# Disable directory listing

As with the [hiding of version and OS identity](Hide-version-and-OS-identity.md), this is not a direct security threat as it only allows the attacker to gain information. 

Add the following configuration in `/etc/apache2/apache2.conf` to disable directory listing globally (server wide):

```
<Directory />
    Options -Indexes 
</Directory>
```

Do not overwrite this inside any directory. Your root directory may look like this:

```
<Directory />
    Options FollowSymLinks
    AllowOverride None
</Directory>
```

AllowOverride None will prevent any accidental changes in `.htaccess` files.

After that you can enable listing per directory by adding the `Indexes` option:

```
<Directory /www/directory>
    Options Indexes FollowSymLinks
    AllowOverride None
</Directory>
```

This enables the generation of `Indexes` only in that folder. 

