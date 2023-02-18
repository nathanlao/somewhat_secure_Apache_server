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
