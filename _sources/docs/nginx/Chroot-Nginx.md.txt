# Chroot Nginx

Create the directory

    # D=/nginx
    # mkdir -p $D

Create mini Linux filesystem

    # mkdir -p $D/etc
    # mkdir -p $D/dev
    # mkdir -p $D/var
    # mkdir -p $D/usr
    # mkdir -p $D/usr/local/nginx
    # mkdir -p $D/tmp
    # chmod 1777 $D/tmp
    # mkdir -p $D/var/tmp
    # chmod 1777 $D/var/tmp
    # mkdir -p $D/lib64

Nginx needs `/dev/null`, `/dev/random`, and `/dev/urandom`. Install these in the just created `/dev/` directory and add the devices with `mknod`. Avoid mounting all of `/dev/` to ensure that, even if the chroot is compromised, an attacker must break out of the `chroot` to access important devices like `/dev/sda1`.   

    # ls -l /dev/{null,random,urandom}
    # /bin/mknod -m 0666 $D/dev/null c 1 3
    # /bin/mknod -m 0666 $D/dev/random c 1 8
    # /bin/mknod -m 0444 $D/dev/urandom c 1 9

Copy nginx files

    # cp -r /usr/share/nginx/* $D/usr/share/nginx
    # cp -r /usr/share/nginx/html/* $D/www
    # cp /usr/bin/nginx $D/usr/bin/
    # cp -r /var/lib/nginx $D/var/lib/nginx

Find and copy libraries. Using `ldd` to list and then `copying` is preferred over hardlinks to make sure that even if an attacker gains write access to the files they cannot destroy or alter the true system files. 

    # ldd /usr/bin/nginx
    linux-vdso.so.1 (0x00007fffc41fe000)
    libpthread.so.0 => /usr/lib/libpthread.so.0 (0x00007f57ec3e8000)
    libcrypt.so.1 => /usr/lib/libcrypt.so.1 (0x00007f57ec1b1000)
    libstdc++.so.6 => /usr/lib/libstdc++.so.6 (0x00007f57ebead000)
    libm.so.6 => /usr/lib/libm.so.6 (0x00007f57ebbaf000)
    libpcre.so.1 => /usr/lib/libpcre.so.1 (0x00007f57eb94c000)
    libssl.so.1.0.0 => /usr/lib/libssl.so.1.0.0 (0x00007f57eb6e0000)
    libcrypto.so.1.0.0 => /usr/lib/libcrypto.so.1.0.0 (0x00007f57eb2d6000)
    libdl.so.2 => /usr/lib/libdl.so.2 (0x00007f57eb0d2000)
    libz.so.1 => /usr/lib/libz.so.1 (0x00007f57eaebc000)
    libGeoIP.so.1 => /usr/lib/libGeoIP.so.1 (0x00007f57eac8d000)
    libgcc_s.so.1 => /usr/lib/libgcc_s.so.1 (0x00007f57eaa77000)
    libc.so.6 => /usr/lib/libc.so.6 (0x00007f57ea6ca000)
    /lib64/ld-linux-x86-64.so.2 (0x00007f57ec604000)

Do not try to copy `linux-vdso.so`: it is not a real library and does not exist in `/usr/lib`. For copying files in `/usr/lib`, try:

    # cp $(ldd /usr/bin/nginx | grep /usr/lib | sed -sre 's/(.+)(\/usr\/lib\/\S+).+/\2/g') $D/usr/lib

    # /lib64/ld-linux-x86-64.so.2 $D/lib
    # cp /usr/lib/libnss_* $D/usr/lib
    # cp -rfvL /etc/{services,localtime,nsswitch.conf,nscd.conf,protocols,hosts,ld.so.cache,ld.so.conf,resolv.conf,host.conf,nginx} $D/etc

    ...

Kill the running nginx

    # killall -9 nginx

Start chrooted nginx

    # /usr/sbin/chroot /nginx /usr/local/nginx/sbin/nginx -t
    # /usr/sbin/chroot /nginx /usr/local/nginx/sbin/nginx 

Check it will start when the system reboots:

    # echo '/usr/sbin/chroot /nginx /usr/local/nginx/sbin/nginx' >> /etc/rc.local

Configuration file

    # cd /nginx/usr/local/nginx/conf/
    # vi nginx.conf

## The automated way for Nginx 

For Nginx there is a script called [n2chroot](https://github.com/doublerebel/nginx-chroot/blob/master/install_files/n2chroot) to copy shared (libs) files to nginx chrooted jail server. This is tested on 64-bit Linux (Redhat and Friends only). It must be edited so that var BASE points to the chroot directory. 

