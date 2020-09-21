# Instructions
#### A simple glue of useful configurations for development environments.

## Linux

### PHP
```
$ wget -q https://packages.sury.org/php/apt.gpg -O- | sudo apt-key add -
$ sudo echo "deb https://packages.sury.org/php/ buster main" | tee /etc/apt/sources.list.d/php.list
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install php php-fpm php7.4-sqlite php7.4-mysql php7.4-curl php7.4-gd php7.4-zip
$ sudo cp /etc/php/7.4/fpm/pool.d/www.conf /etc/php/7.4/fpm/pool.d/${APPNAME}
$ sudo nano /etc/php/7.4/fpm/pool.d/${APPNAME}
    - [www] -> [${APPNAME}]
    - user = ${APPNAME}
    - group = ${APPNAME}
    - listen = /var/run/php7.4-fpm-${APPNAME}.sock
```

### Nginx
```
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install nginx
$ sudo cp ${APPDIRECTORY}/nginx.conf /etc/nginx/conf.d/${APPNAME}.conf
$ sudo nano /etc/nginx/conf.d/${APPNAME}.conf
    - fastcgi_pass unix:/var/run/php/php7.4-${APPNAME}.sock;
    - root /var/www/${APPNAME}
```

### Mysql (MariaDB)
```
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install mariadb-server
$ mariadb -u root -p -h hostserver
$ CREATE USER 'user'@'hostserver' IDENTIFIED BY 'password';
$ CREATE DATABASE databasename;
$ GRANT ALL PRIVILEGES ON databasename.* TO 'username'@'hostserver';
```

### Environment
```
$ sudo adduser ${APPNAME}
$ sudo groupadd ${APPNAME}
$ sudo usermod -aG ${APPNAME} ${APPNAME}
$ sudo chown ${APPNAME} -R /var/www/${APPNAME}
$ sudo chmod 755 -R /var/www/${APPNAME}
$ sudo systemctl restart php7.4-fpm.service
$ sudo systemctl restart nginx.service
$ sudo cp ${APPDIRCTORY} /var/www/${APPNAME}
```


## Windows

### SSL

Install OpenSSL

#### Create a Private Key

We are going to create a private key called rootSSL.key which we will use to issue the new site certificates.
I use the Nginx web server, so I have created a folder called “SSL” in the Nginx windows folder, and that’s where I am going to create this root key.
Open a command prompt in administrator mode and navigate to your newly created SSL folder in the nginx installation folder.
Type in the following command and enter a password for the private key.
```
openssl genrsa -des3 -out rootSSL.key 2048
```

#### Create the Certificate File

In this step, we are going to create a certificate file called rootSSL.pem from the private key we created in the previous step.
Note: you can choose to create a certificate file that lasts for X number of days. We’re going to choose 1024 days in this example, but you can select any amount – the longer, the better.
Type in the following command:
```
openssl req -x509 -new -nodes -key rootSSL.key -sha256 -days 1024 -out rootSSL.pem
```

#### Get Windows to Trust the Certificate Authority (CA)
We are going to use the Microsoft Management Console (MMC) to trust the root SSL certificate.
```
Step 1 – Press the Windows key + R
Step 2 – Type “MMC” and click “OK”
Step 3 – Go to “File > Add/Remove Snap-in”
Step 4 – Click “Certificates” and “Add”
Step 5 – Select “Computer Account” and click “Next”
Step 6 – Select “Local Computer” then click “Finish”
Step 7 – Click “OK” to go back to the MMC window
Step 8 – Double-click “Certificates (local computer)” to expand the view
Step 9 – Select “Trusted Root Certification Authorities”, right-click “Certificates” and select “All Tasks” then “Import”
Step 10 – Click “Next” then Browse and locate the “rootSSL.pem” file we created in step 2
Step 11 – Select “Place all certificates in the following store” and select the “Trusted Root Certification Authorities store”. Click “Next” then click “Finish” to complete the wizard.
```

#### Issuing Certificates for Local Domains
Creating a Local Domain Site
I’m not going to cover setting up the actual site in Nginx or whatever web server you use.

The first step will be to create a local domain.

You do this in your C:\windows\system32\drivers\etc\hosts file.

#### Create a Private Key for the New Domain
We’re going to create a file called “client-1.local.key” which contains the private key information for that domain.

In the same administrator command window type the following:
```
openssl req -new -sha256 -nodes -out client-1.local.csr -newkey rsa:2048 -keyout client-1.local.key -subj "/C=AU/ST=NSW/L=Sydney/O=Client One/OU=Dev/CN=client-1.local/emailAddress=hello@client-1.local"
```
When you are issuing certificates for your own local domains, replace “client-1.local” with your local server domain name.

You can also change the “-subj” parameter to reflect your country, state, location etc.

#### Issue the New Certificate Using the Root SSL Certificate
In the same administrator command window type the following:
```
openssl x509 -req -in client-1.local.csr -CA rootSSL.pem -CAkey rootSSL.key -CAcreateserial -out client-1.local.crt -days 500 -sha256 -extensions "authorityKeyIdentifier=keyid,issuer\n basicConstraints=CA:FALSE\n keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment\n  subjectAltName=DNS:client-1.local"
```
When you are issuing certificates for your own local domains, replace “client-1.local” with your local server domain name.

Enter the password for the root SSL certificate when prompted.

You can see all the files we have created; “client-1.local.crt” and “client-1.local.key” are the files you will need to add to your web server configuration for the local development site.

#### Using the New Local Domain Certificates in Your Web Server

nginx.conf
```
server{
    listen       80;
    server_name  client-1.local;

    # New Lines below
    listen 443 ssl;	
    ssl on;
    ssl_certificate f:/LDE/nginx/SSL/client-1.local.crt;
    ssl_certificate_key f:/LDE/nginx/SSL/client-1.local.key;
}
```
          
