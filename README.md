# somewhat_secure_Apache_server

Step 1. Create a `Linux Server` on `Google Cloud Platform` with the following characteristics: **connection using SSH**
    - A Linux disk image running Ubuntu 20.04 LTS
    - Allows both HTTP and HTTPS traffic
    - 2 vCPU with 2 GB of onboard memory 

## Apache server on Ubuntu Linux setup
Highlights from https://www.linux.com/audience/devops/apache-ubuntu-linux-beginners/

- `Apache HTTP server` powers vast `hosting centers`, and it is also for running small `personal sites`

- `.htaccess` file allows us to make per-directory configurations. No need to use `.htaccess` if you have access to your `Apache configuration files`. You need `.htaccess` if you don't

#### Installing Apache on Linux Ubuntu
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