1. [Create Digital Ocean Droplet with SSH login](#Create-Digital-Ocean-Droplet-with-SSH-login)
1. [Install Server Dependencies](#Install-Server-Dependencies)
1. [Publish your Project to Github](#Publish-your-Project-to-Github)
1. [Hosting Static Files with Digital Ocean Spaces](#Hosting-Static-Files-with-Digital-Ocean-Spaces)
1. [Creating systemd Socket and Service Files for Gunicorn](#Creating-systemd-Socket-and-Service-Files-for-Gunicorn)
1. [DEBUGGING](#DEBUGGING)
1. [Domain Setup](#Domain-Setup)
1. [Create a superuser](#Create-a-superuser)
1. [Finishing up](#Finishing-up)


## Specifications
1. Ubuntu 20.04
1. PostgreSQL
1. Django Channels 2
1. Digital Ocean Spaces (Static files hosted through AWS)
1. Run Django project as WSGI using gunicorn and systemd
1. Configure Nginx to Proxy Pass to Gunicorn (protect from attackers)
1. Configuring the Firewall
1. Redis Install and Config
1. ASGI for Hosting Django Channels
1. Deploying Django Channels with Daphne & Systemd (running the ASGI app)
    1. Starting the daphne service
    1. Writing a bash script that tells daphne to start
    1. Configuring systemd to execute bash script on server boot
1. HTTPS with [letsencrypt](https://letsencrypt.org/)


# Create Digital Ocean Droplet with SSH login

#### Create a new droplet
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/create_droplet.PNG" width="50%" height="50%">
</div>
<br>

#### Droplet configuration
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/digital_ocean_droplet_1.PNG">
</div>
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/digital_ocean_droplet_2.PNG">
</div>
<br>


#### SSH key
Make sure to choose the SSH key option for authentication. Otherwise hackers can simply try passwords to log into your server. This is very bad. Using an SSH key is much, much more secure.

To create an SSH key just click the button "New SSH key" and follow the instructions. **Make sure to save a backup of the private key and public key**. I usually save on an external drive along with on my PC.

<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/digital_ocean_ssh.PNG">
</div>
<br>

#### Finish up
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/digital_ocean_droplet_3.PNG">
</div>
<br>

#### Your IP
Write down your server ip somewhere. You'll need this for logging into your server.

<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/server_ip.PNG">
</div>
<br>

# Log into Droplet with SSH and FTP
Personally I like to use [MobaXterm](https://mobaxterm.mobatek.net/) (it's free) to log into my servers. It's great because you can SSH and FTP from the same window. It's very convenient. 

#### Create a new session
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/mobaxterm_session.PNG">
</div>
<br>

#### Choose SSH
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/ssh_session.PNG">
</div>
<br>

#### SSH Settings
1. Set the server ip
1. set "root" as username
1. Under "Advanced SSH settings":
    1. click "use private key" and choose the location of where you saved your private key.
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/session_settings.PNG">
</div>
<br>

#### Connected
Now connect and it should look like this. There's an FTP on the left and SSH on the right.
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/logged_in.PNG">
</div>
<br>

# Install Server Dependencies
Run these commands in the SSH terminal.

`passwd` Set password for root. I'm not sure what the default is.

`sudo apt update`

`sudo apt install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx curl`

`sudo -u postgres psql`

`CREATE DATABASE django_db;`

`CREATE USER django WITH PASSWORD 'password';`

`ALTER ROLE django SET client_encoding TO 'utf8';`

`ALTER ROLE django SET default_transaction_isolation TO 'read committed';`

`ALTER ROLE django SET timezone TO 'UTC';`

`GRANT ALL PRIVILEGES ON DATABASE django_db TO django;`

`\q`

`sudo -H pip3 install --upgrade pip`

`sudo -H pip3 install virtualenv`

`sudo apt install git-all`

`sudo apt install libgl1-mesa-glx` Resolve cv2 issue

`adduser django`

`su django`

`cd /home/django/`

`mkdir CodingWithMitchChat` You can replace "CodingWithMitchChat" with your project name. 

`cd CodingWithMitchChat`

`virtualenv venv`

`source venv/bin/activate`

`mkdir src`


# Publish your Project to Github
1. Log into Github.com
1. Create a new repository [https://github.com/new](https://github.com/new)
1. Open a cmd prompt to your local project directory
    - Ex: `D:\DjangoProjects\ChatServerPlayground\venv\src`
1. `git init`
1. Update gitignore. I suggest copying mine: [https://github.com/mitchtabian/Codingwithmitch-Chat/blob/master/.gitignore](https://github.com/mitchtabian/Codingwithmitch-Chat/blob/master/.gitignore)
1. `git add .`
1. `git commit -m "first commit"`
1. `git remote add origin https://github.com/mitchtabian/testoinggfmdgmdfk.git` Replace with your git project url.
1. `git push -u origin master`

#### Create a "production" branch
The production branch requires different settings. For example things like static files will be served from AWS via 'Digital Ocean Spaces'. So our static file configuration completely changes compared to development. 

Because of this, I recommend maintaining a "prod" branch. We will deploy the prod branch to the server.

```git
git checkout master
```

```git
git checkout -b prod
```

```git
git push origin prod
```

We will come back to the `prod` branch and make changes to it before we publish.


# Hosting Static Files with Digital Ocean Spaces
1. Create new Digital Ocean Space with the name "open-chat-dev"
    - [https://cloud.digitalocean.com/spaces](https://cloud.digitalocean.com/spaces)
1. Create a API key here [https://cloud.digitalocean.com/account/api/tokens](https://cloud.digitalocean.com/account/api/tokens)
1. Create a folder inside the space and call it "open-chat-static" 

## Update 'prod' branch
Make the necessary changes to the prod branch.

#### settings.py
Top of the file:
```python
from pathlib import Path
from decouple import config

DEBUG = config('DEBUG', default=False, cast=bool)

ALLOWED_HOSTS = ["<ip_from_digital_ocean>",]

ROOT_URLCONF = f'{config("PROJECT_NAME")}.urls'

WSGI_APPLICATION = f'{config("PROJECT_NAME")}.wsgi.application'

ASGI_APPLICATION = f'{config("PROJECT_NAME")}.routing.application'

```

Bottom of the file:
```python

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': config("DB_NAME"),
        'USER': config("DB_USER"),
        'PASSWORD': config("DB_PASSWORD"),
        'HOST': 'localhost',
        'PORT': '',
    }
}

AWS_ACCESS_KEY_ID = config('AWS_ACCESS_KEY_ID')
AWS_SECRET_ACCESS_KEY = config('AWS_SECRET_ACCESS_KEY')
AWS_STORAGE_BUCKET_NAME = config('AWS_STORAGE_BUCKET_NAME')
AWS_S3_ENDPOINT_URL = config('AWS_S3_ENDPOINT_URL')
AWS_S3_OBJECT_PARAMETERS = {
    'CacheControl': 'max-age=86400',
}
AWS_LOCATION = config('AWS_LOCATION')

STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static'),
]

#STATIC_ROOT = os.path.join(BASE_DIR, 'static/')
STATIC_URL = 'https://%s/%s/' % (AWS_S3_ENDPOINT_URL, AWS_LOCATION)
TEMP = os.path.join(BASE_DIR, 'temp')
STATICFILES_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'


EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_HOST_USER = config('EMAIL_HOST_USER')
EMAIL_HOST_PASSWORD = config('EMAIL_HOST_PASSWORD')
EMAIL_PORT = 587
EMAIL_USE_TLS = True
DEFAULT_FROM_EMAIL = 'CodingWithMitch Team <noreply@codingwithmitch.com>'


BASE_URL = "http://<ip_from_digital_ocean>"
```


#### Create settings.ini file in root directory
```
[settings]
DEBUG=False
SECRET_KEY=e9lgp7glzo&n(l3v&jkwhyt8ye*!o=cwh7y6o@b2a^$muup!#1

AWS_ACCESS_KEY_ID=AHPYOBHNGF4FSKE2TXD7
AWS_SECRET_ACCESS_KEY=FJ/i1oP13VJQTj6xZhNnbB6p0jdTGiW0G9va6IkXj58
AWS_STORAGE_BUCKET_NAME=open-chat-xyz-demo
AWS_S3_ENDPOINT_URL=https://nyc3.digitaloceanspaces.com
AWS_LOCATION=open-chat-static

DB_NAME=django_db
DB_USER=django
DB_PASSWORD=password

EMAIL_HOST_USER=<some-email@gmail.com>
EMAIL_HOST_PASSWORD=<password>

PROJECT_NAME=CodingWithMitchChat
```

#### Update directory names
1. Change `ChatServerPlayground` to `CodingWithMitchChat`
    - This is very important. If you don't have the correct directory names the services we run use to run the django application on the server will not work!


#### Remove unnecessary directories
1. Delete `/static_cdn/`
1. Delete `/media/`
1. Delete `/media_cdn/`
1. Delete all migrations
    - Leave the `__init__.py` file. Just delete any other migrations.

#### Update requirements.txt
Make sure you have these installed.
1. `gunicorn psycopg2-binary` (required for postgres)
1. `django-storages` (required for Digital ocean spaces)
1. `boto3` (required for Digital ocean spaces)
1. `python-decouple` (required for settings.ini file)


#### Push code changes to remote
```git
git add .
```

```git
git commit -m "update prod"
```

```git
git push origin prod
```

# Get the Code on the Server
Open MobaXterm and log into your server via SSH.

`su django`

`source venv/bin/activate`

`cd src`

`git init`

`git pull https://github.com/mitchtabian/Codingwithmitch-Chat.git prod` **Or whatever your git url is**

`pip install -r requirements.txt`

`python manage.py collectstatic`


# Check if you can run your project (TEST)
`su root`

`sudo ufw allow 8000`

`su django`

`source venv/bin/activate`

`cd src`

`python manage.py makemigrations`

`python manage.py migrate`

`python manage.py runserver 0.0.0.0:8000`

visit [http://<your_ip_address>:8000/](http://<your_ip_address>:8000/)

`CTRL+C`


# Creating systemd Socket and Service Files for Gunicorn
We've tested to see if the application will run if we run the app manually, but this isn't how we want it to be running. We want the application to run in a service so it automatically starts/restarts when it needs to. Like when we restart the server or it goes down for some reason.

One way you can do this is with gunicorn. Run this command you'll see that gunicorn can run the application:
```
gunicorn --bind 0.0.0.0:8000 CodingWithMitchChat.wsgi
```

visit [http://<your_ip_address>:8000/](http://<your_ip_address>:8000/)

So we just need a service to run that command when the server starts. One way to do that is using [systemd](https://en.wikipedia.org/wiki/Systemd)

`CTRL+C`

#### Configure systemd to execute gunicorn via a `gunicorn.socket` file

`su root`

Navigate to `/etc/systemd/system/`

Create a file named: `gunicorn.socket`

Add the following to the file and save:
```
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```

#### Create gunicorn service to run the WSGI application (the django app)
create new file: `gunicorn.service`

Add the following to `gunicorn.service` and save. **It's very important to copy this exactly as I have. Also your directory structure inside /home/django/ must be EXACTLY the same as mine. Otherwise this service file won't know what project you're talking about.**
```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=django
Group=www-data
WorkingDirectory=/home/django/CodingWithMitchChat/src
ExecStart=/home/django/CodingWithMitchChat/venv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          CodingWithMitchChat.wsgi:application

[Install]
WantedBy=multi-user.target
```

`sudo systemctl start gunicorn.socket`

`sudo systemctl enable gunicorn.socket`


#### Helpful Commands
1. `sudo systemctl daemon-reload`
    - Must be executed if you change the `gunicorn.service` file.
1. `sudo systemctl restart gunicorn`
    - If you change code on your server you must execute this to see the changes take place.
1. `sudo systemctl status gunicorn`
1. `sudo shutdown -r now`
    - restart the server


#### Configure Nginx to Proxy Pass to Gunicorn
We'll be using Nginx as an HTTP Proxy. It helps to protect our website from attackers. You can read more about it here [https://docs.gunicorn.org/en/stable/deploy.html](https://docs.gunicorn.org/en/stable/deploy.html). We need to configure Nginx and gunicorn to work together.

Navigate to `/etc/nginx/sites-available`

Create file `CodingWithMitchChat`
```
server {
    server_name <your_ip_address>;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/django/CodingWithMitchChat/src;
    }
    
     location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
    

}
```


Update Nginx config file at `/etc/nginx/nginx.conf` so we can upload large files (images)
```
http{
	...
	client_max_body_size 10M; 
}
```

#### Configure the Firewall
`sudo ln -s /etc/nginx/sites-available/CodingWithMitchChat /etc/nginx/sites-enabled`

`sudo nginx -t`

`sudo systemctl restart nginx`

`sudo ufw delete allow 8000`

`sudo ufw allow 'Nginx Full'`

`sudo systemctl restart gunicorn`

Set `DEBUG=False` in `settings.ini` if it isn't already.

`service gunicorn restart` (There is no difference between this command and `sudo systemctl restart gunicorn`)

Restart the server `sudo shutdown -r now`

Visit http://<your_ip_address>/


# DEBUGGING
Here are some commands you can use to look at the server logs. **These commands are absolutely crucial to know.** If your server randomly isn't working one day, this is what you use to start debugging.
1. `sudo journalctl` is where all the logs are consolidated to. That's usually where I check.
1. `sudo tail -F /var/log/nginx/error.log` View the last entries in the error log
1. `sudo journalctl -u nginx` Nginx process logs 
1. `sudo less /var/log/nginx/access.log` Nginx access logs
1. `sudo less /var/log/nginx/error.log` Nginx error logs
1. `sudo journalctl -u gunicorn` gunicorn application logs
1. `sudo journalctl -u gunicorn.socket` check gunicorn socket logs



# Domain Setup
If you want a custom domain name (which probably everyone does), this section will take you through how to do that.

#### Purchasing a domain
I like to use [namecheap.com](https://www.namecheap.com/) but it doesn't matter where you buy it from. 

#### Point DNS to Digital Ocean

On the home screen, click the "manage" button on the domain you purchased.
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/namecheap_home.PNG">
</div>
<br>

In the "nameservers" section, select "custom DNS" and point to digital ocean.
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/nameservers.PNG">
</div>
<br>

#### Add the Domain in Digital Ocean

Select your project in digital ocean and click "add domain" on the right.
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/add_a_domain.PNG">
</div>
<br>

Fill in your domain name.
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/add_a_domain_1.PNG">
</div>
<br>

Add the following DNS records. Replace `open-chat.xyx` with your domain name. And you can ignore the CDN.
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/dns_records.PNG">
</div>
<br>

#### Update Nginx config
Earlier we configured Nginx to proxy pass to gunicorn. We need to add the new domain to that configuration.

visit `/etc/nginx/sites-available`

Update `CodingWithMitchChat`
```
server {
    server_name 157.245.134.6 open-chat-demo.xyz www.open-chat-demo.xyz;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/django/CodingWithMitchChat/src;
    }
    
     location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
```

`sudo systemctl reload nginx`

Make sure nginx configuration is still good.

`sudo nginx -t`

#### Update `ALLOWED_HOSTS`

Navigate to `/home/django/CodingWithMitchChat/src/CodingWithMitchChat/`

Update `settings.py` with the domain you purchased. Also make sure your ip is correct.
```
ALLOWED_HOSTS = ["157.245.134.6", "open-chat-demo.xyz", "www.open-chat-demo.xyz"]
```

Apply the changes

`service gunicorn restart`

## TIME TO WAIT...
It can take some time to see your website available at the custom domain. I don't really know how long this will actually take. I waited about an hour and it was working for me.

#### How do you know it's working?
Visiting your domain you should see this **OR you should see your project live and working**.
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/welcome_to_nginx.PNG">
</div>
<br>

# HTTPS (If you have a domain registered and it's working)
**Do not do this step unless you're able to visit your website using the custom domain.** See [How do you know it's working?](How-do-you-know-its-working?)

#### Install certbot
HTTPS is a little more difficult to set up when using Django Channels. Nginx and Daphne require some extra configuring.

`sudo apt install certbot python3-certbot-nginx`

`sudo systemctl reload nginx`

Make sure nginx configuration is still good.
```
sudo nginx -t
```

#### Allow HTTPS through firewall

`sudo ufw allow 'Nginx Full'`

`sudo ufw delete allow 'Nginx HTTP'` Block standard HTTP  


#### Obtain SSL certificate

`sudo certbot --nginx -d <your-domain.whatever> -d www.<your-domain.whatever>`

For me:
```
sudo certbot --nginx -d open-chat-demo.xyz -d www.open-chat-demo.xyz
```

#### Verifying Certbot Auto-Renewal

`sudo systemctl status certbot.timer`

#### Test renewal process
`sudo certbot renew --dry-run`

You should see this
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/verify_certbot.PNG">
</div>
<br>

#### Update CORS in digital ocean
Update for HTTPS in spaces settings
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/CORS.PNG">
</div>
<br>


#### Update settings.py
Set `BASE_URL` variable in `settings.py` to your domain name.


## Update nginx config
We need to tell nginx to allow websocket data to move through port 8001. I'm not really sure how to explain this. I don't understand it fully. Similar to how we allow gunicorn to proxy pass nginx.

Navigate to `/etc/nginx/sites-available`

Update `CodingWithMitchChat`
```
server {

    ...
    
    location /ws/ {
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_redirect off;
        proxy_pass http://127.0.0.1:8001;
    }

    ...
}
```

# Create a superuser
Before you test the server create a superuser.

`su django`

`cd /home/django/CodingWithMitchChat/`

`source venv/bin/activate`

`cd src`

`python manage.py createsuperuser`


# Finishing up
Restart the server and visit your website to try it out. Everything should be working now.

**If you followed my course remember to create a public chat room from the admin with the title "General".**

Thanks for reading and feel free to contribute to this document if you have a better way of explaining things. I am by no means a web expert. 

