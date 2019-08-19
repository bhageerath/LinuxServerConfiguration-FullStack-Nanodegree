# LinuxServerConfiguration-FullStack-Nanodegree
Link to Project [Full Stack Web Developer Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004)

# Project Info 
- **Server IP Address:** 167.71.225.39
- **SSH port:** 2200
- **SSH login username:** grader
- **Application URL:** http://167.71.225.39.xip.io/

## Steps to Set up Server

### Creating the RSA Key Pair

To generate a key pair, run the following command:

   ```console
   $ ssh-keygen
   ```

You now have a public ('udacity.pub') and private key ('udacity') that you can use to authenticate.

### Setting Up a Project and updating the System

 Created Account on [DigtalOcean](https://cloud.digitalocean.com/login).

  Created Droplet

  In the section **Add Your SSH Keys**, paste the content of your public key, `udacity.pub`:
 
   ```
   PasswordAuthentication no
   ```
  IPv4 assigned to me was `167.71.225.39` after creating droplet.

### Logging In as `root` via SSH and Updating the System

```
  $ ssh root@167.71.225.39
```

```
 # apt update
```
```
 # apt upgrade
```

### Change SSH Port to 2200

   ```
   # nano /etc/ssh/sshd_config
   ```

  Change `#Port 22`  to `Port 2200`.

  restart the SSH server
   ```
   # service ssh restart
   ```
   
### Timezone to Use UTC


```
# sudo dpkg-reconfigure tzdata
```

###  Firewall settings

Now we would configure the firewall for SSH (port 2200), HTTP (port 80), and NTP (port 123):

```
# ufw allow 2200/tcp
# ufw allow 80/tcp
# ufw allow 123/udp
# ufw enable
# ufw status
```

### Creating 'grader' User 

```
  # adduser grader
  # usermod -aG sudo grader
  # su - grader
```
### Putting public key in  authroized_keys

```
$ mkdir .ssh
$ chmod 700 .ssh
$ cd .ssh/
$ touch authorized_keys
$ chmod 644 authorized_keys
```

Paste the public key to the server's `authorized_keys` .

### Disabling Root Login

   ```
   $ ssh root@167.71.225.39 -p 2200
   # nano /etc/ssh/sshd_config
   ```

  Change `PermitRootLogin yes` to `PermitRootLogin no`.

   Restart the SSH server:
   ```
   # service ssh restart
   ```
  When you try to log in again as `root`, there will be error :

   ```console
    ssh root@167.71.225.39 -p 2200
    Permission denied (publickey).
   ```

###  Installing Apache Web Server PiP , Git and PostgreSQL


```
$ sudo apt update
$ sudo apt install apache2
$ sudo apt install python-pip
$ sudo add-apt-repository ppa:git-core/ppa
$ sudo apt update
$ sudo apt install git
$ sudo apt install postgresql-10
$ nano /etc/apt/sources.list.d/pgdg.list
```


   And, add the following line to pgdg.list:
   ```
   deb http://apt.postgresql.org/pub/repos/apt/ bionic-pgdg main
   ```

   Import the repository signing key, and update the package lists:

   ```
   $ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
   $ sudo apt update
   ```

### Configuring PostgreSQL

  Log in as the user `postgres` that was automatically created during the installation of PostgreSQL Server:

   ```
   $ sudo su - postgres
   ```

 Open the `psql` shell:

   ```
   $ psql
   ```

 This will open the `psql` shell. Now type the following commands one-by-one:

   ```sql
   postgres=# CREATE DATABASE catalog;
   postgres=# CREATE USER catalog;
   postgres=# ALTER ROLE catalog WITH PASSWORD 'yourpassword';
   postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
   ```

### Installing `mod_wsgi`

   
```
$ sudo apt install libapache2-mod-wsgi
$ sudo service apache2 restart
```

###  Clone Project


   ```
   $ cd /var/www/
   $ sudo mkdir FlaskApp
   $ cd FlaskApp/
   $ sudo git clone https://github.com/bhageerath/ItemCatalog-fullstack-nanodegree.git FlaskApp
   $ cd FlaskApp/
   $ sudo pip install --upgrade Flask SQLAlchemy httplib2 oauth2client requests psycopg2 psycopg2-binary
   ```
   
### Setting Up VirtualHost Config

   ```
   $ sudo nano /etc/apache2/sites-available/FlaskApp.conf
   ```

  Add the following lines to it:

   ```

   <VirtualHost *:80>
      ServerName 167.71.225.39
      ServerAlias 167.71.225.39.xip.io
      ServerAdmin emailid@domain.com
      WSGIScriptAlias / /var/www/FlaskApp/FlaskApp/flaskapp.wsgi
      <Directory /var/www/FlaskApp/FlaskApp/>
          Require all granted
      </Directory>
      Alias /static /var/www/FlaskApp/FlaskApp/static
      <Directory /var/www/FlaskApp/FlaskApp/static/>
          Require all granted
      </Directory>
      ErrorLog ${APACHE_LOG_DIR}/error.log
      LogLevel warn
      CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>

   ```
   Enable the virtual host and restart apache:

   ```
   $ sudo a2ensite FlaskApp
   $ sudo service apache2 restart
   ```

  Creating flaskapp.wsgi File

   ```
   $ cd /var/www/FlaskApp/FlaskApp/
   $ sudo nano flaskapp.wsgi
   ```

Add the following lines to in the created file:

   ```python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0, "/var/www/FlaskApp/FlaskApp/")

   from application import app as application
   ```
  

###  Modify database_setup.py and application.py
Change create engine line in database_setup.py and application.py to:
  ```
  engine = create_engine('postgresql://catalog:yourpassword@localhost/catalog')
  ```
### Update client_secrets.json path and restart Apache server
  Change client_secrets.json path in application.py to:
  ```
    '/var/www/FlaskApp/FlaskApp/client_secrets.json'
   ```
   Restart Apache Server:
  ```
   $ sudo a2dissite 000-default.conf
   $ sudo service apache2 restart
  ```  

   Now application will run at at <http://167.71.225.39.xip.io/>.
 

### Path to check error logs: 

```
$ sudo cat /var/log/apache2/error.log
```

## References

[1] <https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04>

[2] <https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps>

[3] <http://modwsgi.readthedocs.io/en/develop/user-guides/quick-configuration-guide.html>
