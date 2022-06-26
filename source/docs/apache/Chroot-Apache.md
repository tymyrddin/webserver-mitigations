# Chroot Apache

Installing Apache in a `chroot` jail does not make it any more secure in and of itself, but restricts the access of Apache and its child processes to a small subset of the filesystem. It does not prevent a break-in, it contains a potential threat.

* If Apache is compromised, an intruder will have access only to the files within the jail.
* Potentially dangerous scripts do not have access to the server's filesystem.
* The web tree is contained in one area that's easy to back up and move.
* A `chroot` environment can be more difficult to set up, especially when running external software such as Perl, PHP, MySQL, or Python.
* Works only if the entire web tree exists on a single filesystem.

The steps are similar to the ones used for creating a simple jail with bash and ls, that is, building a “virtual root” containing every file the program may need. For Apache it is just slightly more complex: Libraries (C, libssl, libm, libmysqlclient), resolver configuration files (`/etc/nsswitch.conf`, `/etc/resolv.conf`), user files (`/etc/passwd`, `/etc/group`), a separate directory for log files, additional modules needed by the program (for Apache: mod_php and other modules). And then run the program, read the error message(s), copy the missing file, start over. Puzzle mania fun.

Apache runs as `nobody` by default. User `nobody` may be used by many processes, and if it is compromised an intruder will gain access to all processes on your system running under that UID. Configure Apache to run with its own user and group IDs.

    # groupadd apache
    # useradd -c "Apache Server" -d /dev/null -g apache -s /bin/false apache</code>

Create the jail as a mini-version of the Linux filesystem in a (for example) separate partition mounted as `/chroot`, with Apache under a directory named httpd.

    # mkdir /chroot/httpd
    # mkdir /chroot/httpd/dev
    # mkdir /chroot/httpd/lib
    # mkdir /chroot/httpd/etc
    # mkdir -p /chroot/httpd/usr/sbin
    # mkdir /chroot/httpd/usr/lib
    # mkdir /chroot/httpd/usr/libexec
    # mkdir -p /chroot/httpd/var/run
    # mkdir -p /chroot/httpd/var/log/apache
    # mkdir -p /chroot/httpd/home/httpd

Set permissions on the directory structure (for example)

    # chown -R root /chroot/httpd
    # chmod -R 0755 /chroot/httpd
    # chmod 750 /chroot/httpd/var/log/apache/

When Apache is running in the chroot jail, it sees the `/chroot/httpd` directory as the equivalent of `/`. Meaning it cannot access `/dev/null` or `/var/log` on the normal filesystem. Create the null device

    # mknod  /chroot/httpd/dev/null c 1 3
    # chown root.sys /chroot/httpd/dev/null 
    # chmod 666 /chroot/httpd/dev/null

Shut down Apache, run `killall httpd`, and copy the necessary files

    # cp -r /etc/apache /chroot/httpd/etc/    /*configuration files*/
    # cp -r /home/httpd/html /chroot/httpd/home/httpd/    /*Apache DocumentRoot*/
    # cp -r /home/httpd/cgi-bin /chroot/httpd/home/httpd/    /*CGI scripts*/
    # cp /usr/sbin/httpd /chroot/usr/sbin/    /*httpd binary*/
    # cp /usr/sbin/apache* /chroot/usr/sbin/    /*Apache scripts*/
    # cp -a /etc/ssl /chroot/httpd/etc/    /*if mod_ssl, the /etc/ssl directory and
    its contents*/
    # cp -r /usr/libexec/apache /chroot/httpd/usr/libexec/ /*modules from original install*/

Find libraries with

    # ldd /chroot/httpd/usr/sbin/httpd
    /lib/libsafe.so.2 => /lib/libsafe.so.2 (0x40017000)
    libm.so.6 => /lib/libm.so.6 (0x40037000)
    libcrypt.so.1 => /lib/libcrypt.so.1 (0x40059000)
    libdb.so.2 => /lib/libdb.so.2 (0x40086000)
    libexpat.so.0 => /usr/lib/libexpat.so.0 (0x40096000)
    libdl.so.2 => /lib/libdl.so.2 (0x400b6000)
    libc.so.6 => /lib/libc.so.6 (0x400b9000)
    /lib/ld-linux.so.2 => /lib/ld-linux.so.2 (0x40000000)

Copy libraries

    # cp /lib/libsafe* /chroot/httpd/lib/
    # cp /lib/libm* /chroot/httpd/lib/
    # cp /lib/libcrypt* /chroot/httpd/lib/
    # cp /lib/libdb* /chroot/httpd/lib/
    # cp /usr/lib/libexpat* /chroot/httpd/usr/lib/
    # cp /lib/libdl* /chroot/httpd/lib/
    # cp /lib/libc* /chroot/httpd/lib/
    # cp /lib/ld-* /chroot/httpd/lib/

Copy libraries for some standard networking functionality:

    # cp /lib/libnss_compat* /chroot/httpd/lib/
    # cp /lib/libnss_dns* /chroot/httpd/lib/
    # cp /lib/libnss_files* /chroot/httpd/lib/
    # cp /lib/libnsl* /chroot/httpd/lib/

Edit the `/etc/passwd` and `/etc/group` files to contain only entries for the Apache user and group (see first step)

    /etc/passwd:
    apache:x:12347:12348:Apache Server:/dev/null:/bin/false

    /etc/group:
    apache:x:12347:

Copy network configuration files:

    # cp /etc/hosts /chroot/httpd/etc/
    # cp /etc/host.conf /chroot/httpd/etc/
    # cp /etc/resolv.conf /chroot/httpd/etc/
    # cp /etc/nsswitch.conf /chroot/httpd/etc/

And write protect the configuration files:

    # chattr +i /chroot/httpd/etc/hosts
    # chattr +i /chroot/httpd/etc/host.conf
    # chattr +i /chroot/httpd/etc/resolv.conf
    # chattr +i /chroot/httpd/etc/nsswitch.conf
    # chattr +i /chroot/httpd/etc/passwd
    # chattr +i /chroot/httpd/etc/group

`localtime` is a symlink to a file in `/usr/share/zoneinfo`. Find out which file to copy and copy it.

    # ls -l /etc/localtime
    lrwxrwxrwx 1 root root 32 Apr  1 15:48 /etc/localtime -> /usr/share/zoneinfo/Europe/Paris
    # cp /usr/share/zoneinfo/Europe/Paris /chroot/httpd/etc/localtime

Syslogd monitors log files only in `/var/log`. The chrooted httpd daemon will write its logs to `/chroot/httpd/var/log`. Syslogd needs to monitor this directory too. Edit the used startup script, `/etc/rc.d/rc.syslog` or `/etc/rc.d/init.d/syslog`, depending. For `/etc/rc.d/rc.syslog` change `/usr/sbin/syslogd` to `/usr/sbin/syslogd -m 0 -a /chroot/httpd/var/log`    

Create the necessary log files and write-protect them too.

    # touch /chroot/httpd/var/log/apache/access_log
    # touch /chroot/httpd/var/log/apache/error_log
    # chmod 600 /chroot/httpd/var/log/apache/*
    # chattr +a /chroot/httpd/var/log/apache/*

Change the httpd startup script to run the chrooted httpd. Depending, open up `/etc/rc.d/rc.httpd` or `/etc/rc.d/init.d/httpd` and change the command that starts the httpd daemon to read `/usr/sbin/chroot /chroot/httpd/ /usr/sbin/httpd`.

Shut down the httpd daemon and restart the syslog daemon: `/etc/rc.d/rc.syslog restart` (or `/etc/rc.d/init.d/syslog` restart).

Start the chrooted version of Apache with `/etc/rc.d/rc.httpd start` (or `/etc/rc.d/init.d/httpd start`).

Check the daemon is running on the host with the command

    ps -aux | grep httpd

Take the process number from the output of `ps` and run `ls -l /proc/PROC_NUMBER/root/` to see the structure of `/chroot/httpd`

For troubleshooting, use `strace`. Redirecting output of strace to a file named httpd.strace gives an idea of where the problem lies

    # strace chroot /chroot/httpd /usr/sbin/httpd 2> httpd.strace

## The automated way for Apache

`mod_chroot` allows for [running Apache in a chroot jail with no additional files](https://github.com/ZenProjects/Apache-mod-chroot). The `chroot()` system call is performed at the end of startup procedure - when all libraries are loaded and log files open. As of Apache 2.2.10 mod_chroot is installed in httpd by default, all that needs done is referencing ChrootDir in `/etc/httpd/conf/httpd.conf`.

The apache module [mod_unixd](https://httpd.apache.org/docs/2.4/mod/mod_unixd.html) offers the chroot function in Apache 2.4, is part of the Apache core modules and is compiled statically into the Apache binary.

