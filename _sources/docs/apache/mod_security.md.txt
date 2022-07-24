# Install and use mod_security

## Installation
[Compile and embed the ModSecurity module](https://www.netnea.com/cms/apache-tutorial-6_embedding-modsecurity/) or install from repository:

    # apt-get install libapache2-modsecurity

Check it was loaded:

    # apachectl -M | grep --color security

Rename the recommended-labeled configuration file:

    # mv /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf

Restart Apache. 

## Usage

A new logfile named `/var/log/apache2/modsec_audit.log` has been created. Check that the default configuration file is not set to `DetectionOnly`, which logs requests according to rule matches and doesn't block anything. Edit the `/etc/modsecurity/modsecurity.conf` file to change it if need be (set it to `On`). Some possible changes:

    # Prevent path traversal (..) attacks
    SecFilter "../"

    # Weaker XSS protection but allows common HTML tags
    SecFilter "<[[:space:]]*script"

    # Prevent XSS atacks (HTML/Javascript injection)
    SecFilter "<(.|n)+>"

    # Very crude filters to prevent SQL injection attacks

    SecFilter "delete[[:space:]]+from"
    SecFilter "insert[[:space:]]+into"
    SecFilter "select.+from"
    SecFilter "drop[[:space:]]table"

    # Protecting from XSS attacks through the PHP session cookie
    SecFilterSelective ARG_PHPSESSID "!^[0-9a-z]*$"
    SecFilterSelective COOKIE_PHPSESSID "!^[0-9a-z]*$"

Restart Apache. 

## Configuration resources

Have a look at the [OWASP ModSecurity Core Rule Set Project](https://www.owasp.org/index.php/Category:OWASP_ModSecurity_Core_Rule_Set_Project) (CRS) for making more secure changes. The CRS aims to protect web applications from a wide range of attacks, with a minimum of false alerts.

