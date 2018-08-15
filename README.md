# Udacity Linux Server Configuration

Goal: Take an Amazon Ubuntu VM and prepare it to host my catalog project.
This includes applying all system updates, installing web server, python,
database, access controls, firewall config, and other software.

I've checked to ensure every criteria listed in the rubric is met. 

## Amazon Server Details

IP address: 18.191.1.245, SSH port: 2200, URL: http://18.191.1.245.xip.io/

## Resources/Credits
* Article on setting up WSGI for Flask https://www.apptic.me/blog/setting-up-flask-with-red-hat-on-aws.php
* Udcacity Courses for user management and firewall configuration with UFC
* Stack entry on doing all the distribution updates so MOTD is clear at login. https://askubuntu.com/questions/196768/how-to-install-updates-via-command-line

## Software Added for Project

The following were installed with sudo apt-get install:

* Apache2 (apache2)
* PostgreSQL (postgresql)
* WSGI Support (libapache2-mod-wsgi-py2 & libapache2-mod-wsgi)

The following were installed with pip install:

* sqlalchemy 
* oauth2client 
* requests 
* httplib2
* pyscopg2-binary
* Flask-GoogleLogin

## Configuration

### Update all system packages

```sh
sudo apt-get update
sudo apt-get upgrade 
sudo reboot
sudo apt-get dist-upgrade
sudo reboot
```

The message of the day should show no updates pending after login.

### Configure Firewall 
These steps are done using UFW. It's important to make sure
your Amazon cloud configuration has port 2200 and 123 open as well.

Block all incoming requests
```sh
sudo ufw default deny incoming
```

Allow all outgoing requests
```sh
sudo ufw default allow outgoing
```

Allow connections for SSH on TCP port 2200
```sh
sudo ufw allow 2200/tcp
```

Allowing incoming for HTTP 
```sh
sudo ufw allow www
```

Allowing incoming for NTP. Rubric has no instructions on why, but it's open. Ubuntu's default install now uses timesyncd, not ntp. `It's not running since not in the rubric - but opening the port is.`

```sh
sudo ufw allow ntp
```

Enable ufw, check status down
```sh
sudo ufw enable
sudo ufw status
```

### Setup access for grader

#### Create user `grader` 

```sh
sudo adduser grader
```

#### Give grader sudo permission 
Create following new file with content below:

grader ALL=(ALL) NOPASSWD:ALL

```sh
sudo vi /etc/sudoers.d/grader
```

### Set up SSH Authentication for grader (ubuntu user has AWS key)

First run ssh-keygen (or other tool) to generate SSH key pairs.
Copy content of .pub file to the clipboard

```sh
# Run LOCAL machine, no passpharse with default directory 
# and "grader" at the end

ssh-keygen 
```

Configure public key on server for grader (Run on the Server)
```sh
sudo su grader
mkdir .ssh
touch .ssh/authorized_keys
vi .ssh/authorized_keys
chmod 700 .ssh
chmod 644 .ssh/authorized_keys 
```

### Configure SSH and root login

Open SSH config file
```sh
sudo vi /etc/ssh/sshd_config
```
Change Port 22 to 2200

### Ensure remote login of the root disabled

#### SSH Disable
Open SSH config file

```sh
sudo vi /etc/ssh/sshd_config
```

Change PermitRootLogin to  no
#### Shell Disable root
Open /etc/passwd and change root shell to `nologin`
```sh
sudo vi /etc/passwd
# root:x:0:0:root:/root:/usr/sbin/nologin
```

#### Force SSH Logins Only (should be default for AWS)

Make sure PasswordAuthentication has value of `no`
```sh
vi /etc/ssh/sshd_config
```
#### Restart SSH service
```sh
sudo service ssh restart
```

### Install Apache to serve mod_wsgi flask application

Install required packages
```sh
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi
```

Update last line of `/etc/apache2/sites-enabled/000-default.conf` to use the 
wsgi configuration file in the repo. Add the following line right before the closing `</VirtualHost>` line:
```sh
WSGIScriptAlias / /var/www/html/catalog/example.wsgi
```

### Install and configure PostgreSQL

```sh
sudo apt-get install postgresql
```

#### Create a new user named catalog with access rights

Change to postgres user
```sh
sudo -i -u postgres
```

Create new dastbase user `catalog`
```sh
    $ sudo -u postgres createuser -P catalog
    $ sudo -u postgres createdb -O catalog catalog
    $ sudo -u postgres -i
    $ psql
    # \c catalog
    # REVOKE ALL ON SCHEMA public FROM public;
    # GRANT ALL ON SCHEMA public TO catalog;
    #\du
    #\q
    exit
```

### Verify git install, clone repo
NOTE: Since we are mapping the root level to our catalog application the 
.git directory is not accessible to web users.

Verify/Install git
```sh
sudo apt-get install git
```
Clone Repo
```sh
sudo git clone https://github.com/udacitygly/catalog.git
```

Install application dependences
```sh
sudo -h pip install httplib2
sudo -h pip install oauth2client
sudo -h pip install requests
sudo -h pip install psycopg2.binary
sudo -h pip install flask
sudo -h pip install sqlalchemy
```

Restart Apache
```sh
sudo apachectl restart
```

## Conclusion
At this point the URL listed at top and grader access should work properly.
