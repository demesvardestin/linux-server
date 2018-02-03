#### Server Details

 IP address: 18.216.134.208 
 SSH Port: 2200 
 URL: http://18.216.134.208
 
 
#### Getting Started

 First begin by setting up your Amazon Lightsail instance:
    - Choose OS Only
    - Select Ubuntu
 Upon creating your instance, opt to set a custom ssh key-pair to login with and
 download it on your local machine. Then set permissions on the .pem file using
 
 ```
 chmod 600 yourdownloadedpemfile.pem
 ```
 Double check your ability to log into your instance from your local machine:
 
 ```
 ssh ubuntu@your.public.ip.address -i yourdownloadedpemfile.pem
 ```
#### Setting up the server

 Update all packages:
 
 ```
 sudo apt-get update  
 sudo apt-get upgrade
 ```
 Create a new user:
 
 ```
 sudo adduser grader
 ```
 
 After setting up the grader details, proceed to give grader sudo access:
 
 ```
 sudo visudo
 ```
 
 Below the line with the following:
 
 ```
 root  ALL=(ALL:ALL) ALL
 ```
 Add the same line, but replaec root with grader.
 
#### Configuring the firewall (part 1)

 Head to your instance dashboard on Lightsail, and go to the 'networking' tab.
 Edit the Firewall section to add a custom rule with port 2200. Then edit
 your sshd config file with the following command:
 
 ```
 sudo nano /etc/ssh/sshd_config
 ```
 Comment out ``` Port 22``` so that your file looks like this:
 
 ```
    # What ports, IPs and protocols we listen for
    # Port 22
    Port 2200
    
    # Authentication:
    LoginGraceTime 120
    #PermitRootLogin prohibit-password
    PermitRootLogin no
    StrictModes yes
    
    # Change to no to disable tunnelled clear text passwords
    PasswordAuthentication No
 ```
 You should now reload your ssh to reflect the changes made:
 
 ```
 sudo service ssh restart
 ```
 
#### Creating your ssh key pairs
 
 On your LOCAL MACHINE, generate a new key pair with this command:
 
 ```
 ssh-keygen
 ```
 
 You will be prompted to input a destination for the file. It's good practice
 to store it in the directory example that shows in the prompt (typicall the
 .ssh folder). You will be prompted to select a passphrase, which is optional.
 But if you do set one, you will have to remember it for future login attempts.
 Now switch to the grader account with ``` sudo su grader```. Create a new .ssh
 directory and cd into it``` mkdir .ssh && cd .ssh```. Create a new file called
 'authorized keys' in that folder, and paste the content of the public key that
 you generated earlier (yourdownloadedpem.pub). You should now be able to log
 into the grader account from your local machine:
 
 ```
 ssh grader@your.public.ip.address -i ~/.ssh/yoursshkeypairfile
 ```
 
#### Configuring the firewall (part 2)
 You will now need to enable certain ports in your firewall. First start by
 disabling everything, then enabling outgoing requests and allowing specific
 ports.
 
 ```
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    sudo ufw allow 2200/tcp
    sudo ufw allow 80/tcp or sudo ufw allow www (both work)
    sudo ufw allow 123/udp
 ```
 
 Then enable those changes with ``` sudo ufw enable```. Check if the changes
 were reflected with ``` sudo ufw status```. If the changes were made, you
 should see a total of 6 lines, 2 for each enabled port.
 
 Configure your timezone
 
 ```
 sudo dpkg-reconfigure tzdata
 ```
 
 Select 'None of the above', and in the next window, select UTC.
 
#### Setting up Apache and other libraries

 Install apache with ``` sudo apt-get install apache2```, then install wsgi:
 ``` sudo apt-get install libapache2-mod-wsgi python-dev```. Wsgi allows your
 app to be read and interpreted by the Apache server.
 Next step is to install Git, in order to download your app from your Github
 (or any other Git service) repository
 
 ``` sudo apt-get install git```
 Head over to Github and copy the clone link for your app. Then head back to
 your terminal, CD into the www directory ``` cd /var/www```, and clone your
 app there: ``` sudo git clone <your clone link>```. Setup your WSGI file
 within your app directory ``` sudo nano /var/www/<your app folder>/app.wsgi```
 Fill it with the following:
 
 ```
    # WSGI file
    
    import sys
    import logging
    
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, '/var/www/itemsCatalog/vagrant/catalog')
    
    from application import app as application
    application.secret_key='super_secret_key'
 ```
 Note: the line with ``` from application``` assumes that your app folder has
 a file named 'application.py'. If your file is named, for example, 'catalog.py',
 then this line would become ``` from catalog import app as application```.
 Next, edit your .conf file (the apache/wsgi configuration file):
 
 ```
 sudo nano /etc/apache2/sites-available/000-default.conf
 ```
 Fill it with the following:
 
 ```
<VirtualHost *:80>
    # Server details
    ServerName  your.public.ip.address
    ServerAdmin webmaster@localhost (or an actual email)
    
    #Location of the items-catalog WSGI file
    WSGIScriptAlias / /var/www/itemsCatalog/vagrant/catalog/itemsCatalog.wsgi
    
    #Allow Apache to serve the WSGI app from our catalog directory
    <Directory /var/www/your-app-folder>
        Order allow,deny
        Allow from all
    </Directory>
    
    #Allow Apache to deploy static content
    <Directory /var/www/your-app-folder/static>
        Order allow,deny
        Allow from all
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
 ```
 
#### Setup Flask and other Python libs
 
 ```
 sudo apt-get install python-pip python-flask python-sqlalchemy python-psycopg2
 sudo pip install oauth2client requests httplib2
 ```
 
 Postgresql
 
 ```
 sudo apt-get install postgresql
 ```
 
 Set your database table, make sure to change the create engine parameters in
 your python scripts to reflect the changes.
 
#### Acknowledgements

 [Amazon Lightsail](http://amazonlightsail.com)
