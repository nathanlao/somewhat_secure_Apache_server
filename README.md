# somewhat_secure_Apache_server

**Goals**: Installing a self-signed SSL certificate, and creating groups and permissions for the files 

**Server/Environment**: Create a `Linux Server` on `Google Cloud Platform` with the following characteristics: **connection using SSH**
- A Linux disk image running Ubuntu 20.04 LTS
- Allows both HTTP and HTTPS traffic
- 2 vCPU with 2 GB of onboard memory 

## Apache server on Ubuntu Linux setup
Highlights from https://www.linux.com/audience/devops/apache-ubuntu-linux-beginners/

- `Apache HTTP server` powers vast `hosting centers`, and it is also for running small `personal sites`

- `.htaccess` file allows us to make per-directory configurations. No need to use `.htaccess` if you have access to your `Apache configuration files`. You need `.htaccess` if you don't

### Installing Apache on Linux Ubuntu
1. Install apache2 from apt

        sudo apt update

        sudo apt install apache2

2. To check Apache status

        sudo systemctl status apache2

        sudo systemctl stop apache2

        sudo systemctl start apache2

    Point your web browser to http://localhost or `http://<external IP address>`  to verify that it’s running

3. Apache config files on Linux

        cd /etc/apache2/
    
    List of **configuration files**:

        apache2.conf                
        conf-available              
        conf-enabled                
        envvars
        magic
        mods-available
        mods-enabled
        ports.conf
        sites-available
        sites-enabled

## Installing self-signed SSL certificate on Apache
Highlights from https://www.linux.com/training-tutorials/creating-self-signed-ssl-certificates-apache-linux/

Set up your Apache web server to use `SSL`, so that your site URL is `https://` and not `http://`, and connect to the server securely

**Note that**: we will see an `untrusted connection warning` because `cert` is self-signed and the browser doesn’t know your server.

1. Make sure Apache has `SSL` enabled

        sudo a2enmod ssl

2. Generate a `certificate signing request (CSR)`

        sudo openssl req -new > new.ssl.csr

    -  `PEM pass phrase`: refers to the passphrase used to `encrypt` a private key file in the PEM format

3. Generate a `key` and `self-signed certificate`

    - extract the private key from privkey.pem
                
            sudo openssl rsa -in privkey.pem -out new.cert.key

    - generate a self-signed SSL certificate using the OpenSSL

            sudo openssl x509 -in new.cert.csr -out new.cert.cert -req -signkey new.cert.key -days NNN

4. Copy the certificate and keys we’ve generated

        sudo cp new.cert.cert /etc/ssl/certs/server.crt
        sudo cp new.cert.key /etc/ssl/private/server.key

5. Tell Apache about the certificate

    - configuration files located in `/etc/apache2/sites-available/`
    - dafault files:
        1. the `default configuration file` that is used when `no other virtual hosts` are defined
                
                000-default.conf 

        2. the `default configuration file` that is used when `SSL` is enabled on the server       
            
                default-ssl.conf

6. Modify the VirtualHosts to use the certificate
- `https` should listen for `port 443`

        <VirtualHost *:443>
            ServerAdmin <email address here>
            DocumentRoot /var/www/html

            ErrorLog ${APACHE_LOG_DIR}/error.log
            CustomLog ${APACHE_LOG_DIR}/access.log combined

            SSLEngine on
            SSLOptions +StrictRequire
            SSLCertificateFile /etc/ssl/certs/server.crt
            SSLCertificateKeyFile /etc/ssl/private/server.key
        </VirtualHost>

7. Restart Apache and test

        sudo systemctl restart apache2

## Authentication with groups/files in Linux
Highlights from https://httpd.apache.org/docs/2.4/howto/auth.html

**Goal**: If you have information on your web site that is sensitive or intended for `only a small group of people`, the `authentication` techniques will help you make sure that the people that see those pages are the people that you wanted to see them.

A basic `authentication` can be set up by using a `.htaccess` file (stored in each folder) to override the `defaults` (which is no authentication)

1. we need to have a `server configuration` that `permits` putting `authentication directives` in these files. This is done with the `AllowOverride` directive

        AllowOverride AuthConfig

    - Ensure modules `mod_authn_core` and `mod_authz_core` loaded by the `apache2.conf` configuration file 

        sudo apache2ctl -M

2. create a `password file` for protecting a directory on your server

- **For 1 user only**:

    put the password file(s) in /usr/local/apache2/passwd, this password file should be placed `somewhere not accessible from the web`

    To create the file, type:

        sudo htpasswd -c /usr/local/apache2/passwd/passwords usr1
    
    using an `.htaccess` file under the web folder:

        AuthType Basic
        AuthName "Restricted Files"
        # (Following line optional)
        AuthBasicProvider file
        AuthUserFile "/usr/local/apache2/passwd/passwords"
        Require user usr1

- **For groups of users**:

    create a `group file` that associates `group names` with a list of `users` in that group

        sudo vim grp1

    Format in grp1

        grp1: usr1 usr2

    To add a user to your `already existing password file`:

        htpasswd /usr/local/apache/passwd/passwords usr2

    modify `.htaccess` file under the web folder:

        AuthType Basic
        AuthName "By Invitation Only"
        # Optional line:
        AuthBasicProvider file
        AuthUserFile "/usr/local/apache2/passwd/passwords"
        AuthGroupFile "/usr/local/apache/passwd/grp1"
        Require group grp1

That's it! **Note that**: 
    - the `limitation` of basic authentication: the web may asks only once for the authentication since the brower caches the username and password, you can test it in `incognito mode`

## Tutorial Sources

- Vim for beginners: https://linuxconfig.org/vim-tutorial
- Apache on Ubuntu for beginners: https://www.linux.com/audience/devops/apache-ubuntu-linux-beginners/
- Installing self-signed certificates: https://www.linux.com/learn/creating-self-signed-ssl-certificates-apache-linux
- Basic Authentication with user groups in Linux: http://httpd.apache.org/docs/2.4/howto/auth.html