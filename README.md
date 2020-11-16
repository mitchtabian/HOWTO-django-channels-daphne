1. [What's Covered in this Document](#Whats-Covered-in-this-Document)
1. [Create Digital Ocean Droplet with SSH login](#Create-Digital-Ocean-Droplet-with-SSH-login)
1. [Install Server Dependencies](#Install-Server-Dependencies)
1. [Publish your Project to Github](#Publish-your-Project-to-Github)
1. [Hosting Static Files with Digital Ocean Spaces](#Hosting-Static-Files-with-Digital-Ocean-Spaces)
1. [Creating systemd Socket and Service Files for Gunicorn](#Creating-systemd-Socket-and-Service-Files-for-Gunicorn)
1. [DEBUGGING](#DEBUGGING)
1. [Install and Setup Redis](#Install-and-Setup-Redis)
1. [ASGI for Hosting Django Channels as a Separate Application](ASGI-for-Hosting-Django-Channels-as-a-Separate-Application)
1. [Deploying Django Channels with Daphne & Systemd](#Deploying-Django-Channels-with-Daphne-&-Systemd)
1. [Starting the daphne Service when Server boots](#Starting-the-daphne-Service-when-Server-boots)
1. [Domain Setup](#Domain-Setup)
1. [Create a superuser](#Create-a-superuser)
1. [Finishing up](#Finishing-up)
1. [FAQ](#FAQ)
1. [References](#References)


# What's Covered in this Document
Everything involved in publishing a django website equipped with WebSockets using Django Channels. 

I use [Digital Ocean](https://m.do.co/c/c87161ed324c) as the hosting provider. They have amazing products, documentation and customer support. I highly recommend them. Get $100 free with this referral link: [Get $100 Free some Digital Ocean](https://m.do.co/c/c87161ed324c).

**Keep in mind** this document is meant to be followed after watching my course where I show you how to build a real-time chat website. You can check out the course here: [Real-time Chat Messenger course](https://codingwithmitch.com/courses/real-time-chat-messenger/).

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

#### manage.py
```
#!/usr/bin/env python
"""Django's command-line utility for administrative tasks."""
import os
import sys
from decouple import config


def main():
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', f'{config("PROJECT_NAME")}.settings')
    try:
        from django.core.management import execute_from_command_line
    except ImportError as exc:
        raise ImportError(
            "Couldn't import Django. Are you sure it's installed and "
            "available on your PYTHONPATH environment variable? Did you "
            "forget to activate a virtual environment?"
        ) from exc
    execute_from_command_line(sys.argv)


if __name__ == '__main__':
    main()

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


#### Update header.html
The WebSockets will be communicating through port 8001 (we will configure this later). So make sure in all the Javascript WebSockets you are referencing port 8001.
```javascript
var ws_path = ws_scheme + '://' + window.location.host + ":8001/"; // PRODUCTION
````

#### Update base.html
I forgot to add jQuery. You can get it here [https://cdnjs.com/libraries/jquery](https://cdnjs.com/libraries/jquery). Or just add this to `base.html` in the "body" section. **Do not get it from https://getbootstrap.com/**. They have the *slim* version which is not what we want.
```html
<!-- jquery -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.5.1/jquery.min.js" />
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.5.1/jquery.js" integrity="sha512-WNLxfP/8cVYL9sj8Jnp6et0BkubLP31jhTG9vhL/F5uEZmg5wEzKoXp1kJslzPQWwPT1eyMiSxlKCgzHLOTOTQ==" crossorigin="anonymous"></script>
```

#### Add default_profile_image.png to `/static/`
Inside `/media_cdn/` is a file named `default_profile_image.png`. Move that into `/static/`.


#### Remove unnecessary directories
1. Delete `/static_cdn/`
1. Delete `/media/`
1. Delete `/media_cdn/`
1. Delete all migrations
    - Leave the `__init__.py` file. Just delete any other migrations.

#### Update requirements.txt
Make sure to copy my `requirements.txt` file so you have all the necessary dependencies: [requirements.txt](https://github.com/mitchtabian/Codingwithmitch-Chat/blob/prod/requirements.txt).

This `requirements.txt` file has some extra dependencies that we didn't have in development. They can all be installed manually using pip but I added them to `requirements.txt` so you wouldn't have to. This is what was added:
1. `gunicorn psycopg2-binary` (required for postgres)
1. `django-storages` (required for Digital ocean spaces)
1. `boto3` (required for Digital ocean spaces)
1. `python-decouple` (required for settings.ini file)


#### A Problem with how Django interprets static files in javascript
This is a weird thing that happens if you set static files in javascript with django. HTML symbols get translated into their respective code. 

Example
```
& becomes &amp;

Therefore 

https://<some-domain>/&Signature=3454v535435

Becomes

https://<some-domain>/&amp;Signature=3454v535435
```

This is a problem because the image will not load if the `&` symbol becomes`&amp;`. So we need need to fix a couple files for production.

##### `public_chat.html`
Change 
```javascript
profileImage.src = "{% static 'codingwithmitch/dummy_image.png' %}"
```
To
```javascript
profileImage.src = "{% static 'codingwithmitch/dummy_image.png' %}".replace(/&amp;/g, "&")
```

`.replace(/&amp;/g, "&")` means: "Find all the occurances of '&amp;' and replace it with '&'."


##### `account.html`
Change 
```html
<img class="d-block border border-dark rounded-circle img-fluid mx-auto profile-image" alt="codingwithmitch logo" id="id_profile_image" src="{{profile_image}}">
```
To
```html
<img class="d-block border border-dark rounded-circle img-fluid mx-auto profile-image" alt="codingwithmitch logo" id="id_profile_image" src="{% static 'codingwithmitch/dummy_image.png' %}">
```

And down at the bottom make sure to preload the image.
```javascript
preloadImage("{{profile_image|safe}}", 'id_profile_image')
````


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

**At this point you can check in the Digital Ocean spaces console and you should see the static files have been placed there.**
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/after_collect_static.PNG">
</div>
<br>


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



# Install and Setup Redis
Redis is used as a kind of "messaging queue" for django channels. Read more about it here [https://channels.readthedocs.io/en/stable/topics/channel_layers.html?highlight=redis#redis-channel-layer](https://channels.readthedocs.io/en/stable/topics/channel_layers.html?highlight=redis#redis-channel-layer)

`sudo apt install redis-server`

Navigate to `/etc/redis/`

open `redis.conf`

`CTRL+F` to find 'supervised no'

change 'supervised no' to 'supervised systemd'

`SAVE`

`sudo systemctl restart redis.service`

`sudo systemctl status redis`

Should see this:
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/redis_status.PNG">
</div>
<br>

`CTRL+C` to exit.

`sudo apt install net-tools`

Confirm Redis is running at 127.0.0.1. Port should be 6379 by default.

`sudo netstat -lnp | grep redis`

`sudo systemctl restart redis.service`


# ASGI for Hosting Django Channels as a Separate Application
From the Django channels docs:
> ASGI (Asynchronous Server Gateway Interface), is the specification which Channels are built upon, designed to untie Channels apps from a specific application server and provide a common way to write application and middleware code.

`su django`

Create file named `asgi.py` in `/home/django/CodingWithMitchChat/src/CodingWithMitchChat` with this command:

`cat > asgi.py` 'django' must be the owner of this file.

Paste in the following:
```
"""
ASGI entrypoint. Configures Django and then runs the application
defined in the ASGI_APPLICATION setting.
"""

import os
import django
from decouple import config
from channels.routing import get_default_application

os.environ.setdefault("DJANGO_SETTINGS_MODULE", f'{config("PROJECT_NAME")}.settings')
django.setup()
application = get_default_application()

```

`CTRL+D` to save.

You can open the file to confirm everything looks good.

`ls -l` to check ownership. `django` needs to be the owner.
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/CodingWithMitchChat_ownership.PNG">
</div>
<br>

# Deploying Django Channels with Daphne & Systemd
Gunicorn is what we use to run the WSGI application - which is our django app. To run the ASGI application we need something else, an additional tool. **[Daphne](https://github.com/django/daphne)** was built for Django channels and is the simplest. We can start daphne using a systemd service when the server boots, just like we start gunicorn and then gunicorn starts the django app.

Here are some references I found helpful. The information on this is scarce:
1. [https://channels.readthedocs.io/en/latest/deploying.html](https://channels.readthedocs.io/en/latest/deploying.html)
1. [https://stackoverflow.com/questions/50192967/deploying-django-channels-how-to-keep-daphne-running-after-exiting-shell-on-web](https://stackoverflow.com/questions/50192967/deploying-django-channels-how-to-keep-daphne-running-after-exiting-shell-on-web)

`su root`

`apt install daphne`

Navigate to `/etc/systemd/system/`

Create `daphne.service`. Notice the port is `8001`. This is what we need to use for our `WebSocket` connections in the templates.
```
[Unit]
Description=WebSocket Daphne Service
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/home/django/CodingWithMitchChat/src
ExecStart=/home/django/CodingWithMitchChat/venv/bin/python /home/django/CodingWithMitchChat/venv/bin/daphne -b 0.0.0.0 -p 8001 CodingWithMitchChat.asgi:application
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

`systemctl daemon-reload`

`systemctl start daphne.service`

`systemctl status daphne.service`

You should see something like this. If you don't, go back and redo this section. Check that your filepaths are all **exactly the same as mine in `daphne.service`**. That is the #1 reason people have issues.
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/daphe_status.PNG">
</div>
<br>

`CTRL+C`

# Starting the daphne Service when Server boots
With gunicorn and the WSGI application, we created a `gunicorn.socket` file that tells gunicorn to start when the server boots (at least this is my understanding). I couldn't figure out how to get this to work for daphne so instead I wrote a bash script that will run when the server boots. 

#### Create the script to run daphne
Navigate to `/root`

create `boot.sh`
```
#!/bin/sh
sudo systemctl start daphne.service
```

Save and close.

Might have to enable it to be run as a script (not sure if this is needed)
`chmod u+x /root/boot.sh`

If you want to read more about shell scripting, I found this helpful:
[https://ostechnix.com/fix-exec-format-error-when-running-scripts-with-run-parts-command/](https://ostechnix.com/fix-exec-format-error-when-running-scripts-with-run-parts-command/).


#### Tell systemd to run the bash script when the server boots

Navigate to `/etc/systemd/system`

create `on_boot.service`
```
[Service]
ExecStart=/root/boot.sh

[Install]
WantedBy=default.target
```
Save and close.

`systemctl daemon-reload`

##### Start it
`sudo systemctl start on_boot` 

##### Enable it to run at boot
`sudo systemctl enable on_boot` 

##### Allow daphne service through firewall
`ufw allow 8001` 

##### Restart the server
`sudo shutdown -r now`

##### Check the status of `on_boot.service`
`systemctl status on_boot.service`

Should see this. If not, check logs: `sudo journalctl -u on_boot.service`
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/on_boot_service_status.PNG">
</div>
<br>

##### Check if the daphne service started when the server started:
`systemctl status daphne.service`

Should see this. If not, check logs: `sudo journalctl -u daphne.service`
<div class="row  justify-content-center">
  <img class="img-fluid text-center" src = "https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/daphne_service_status.PNG">
</div>
<br>

#### Where are the logs?
journalctl is my general go-to. You can filter specifically for a service like this:
```
sudo journalctl -u on_boot.service // for on_boot.service
sudo journalctl -u daphne.service // for daphne.service
```

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

## Update `daphne.service`
Tell daphne how to access our https cert.

Navigate to `/etc/systemd/system`

Update `daphne.service`
```
[Unit]
Description=WebSocket Daphne Service
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/home/django/CodingWithMitchChat/src
ExecStart=/home/django/CodingWithMitchChat/venv/bin/python /home/django/CodingWithMitchChat/venv/bin/daphne -e ssl:8001:privateKey=/etc/letsencrypt/live/open-chat-demo.xyz/privkey.pem:certKey=/etc/letsencrypt/live/open-chat-demo.xyz/fullchain.pem CodingWithMitchChat.asgi:application  
Restart=on-failure

[Install]
WantedBy=multi-user.target
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


# FAQ
Here are some things I wish I knew when doing this for the first time.

### If you change a file or pull a code update to the project, do you need to do anything?
Yes.

If you only change code that is *not related to django channels* then you only need to run `service gunicorn restart`.

But if you change any code related to django channels, **then you must also restart the daphne service**: `service daphne restart`.

To be safe, I always just run both. It can't hurt.
```
service gunicorn restart
service daphne restart
```
<br>

### Service Status Errors
Throughout this document we periodically check the status of the services that we set up. Things like:
1. `sudo systemctl status gunicorn`
1. `sudo systemctl status redis`
1. `systemctl status daphne.service`
1. `systemctl status on_boot.service`
1. `sudo systemctl status certbot.timer`

If any of these fail, it's not going to work and you've done something wrong. The most common problem is the directory structure does not match up. For example you might use `/home/django/django_project/src/` instead of `/home/django/CodingWithMitchChat/src/`. You need to look very carefully at your directory structures and make sure the naming is all correct and correlates with the `.service` files you build. 

When you make a change to a `.service` file, **Always run `sudo systemctl daemon-reload`**. Or to be safe, just restart the damn server `sudo shutdown -r now`. Restarting the server is the safe way, but also the slowest way. 


### CORS error in web console
You are getting an error in web console saying: "No 'Access-Control-Allow-Origin' header is present on the requested resource".

Fix this by adding CORS header in spaces settings.
See [This image](https://github.com/mitchtabian/HOWTO-django-channels-daphne/blob/master/images/CORS.PNG) for the configuration.

# References
1. [https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-18-04](https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-18-04)
1. [https://channels.readthedocs.io/en/latest/](https://channels.readthedocs.io/en/latest/)
1. [https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-20-04)
1. [https://www.digitalocean.com/community/tutorials/how-to-set-up-object-storage-with-django](https://www.digitalocean.com/community/tutorials/how-to-set-up-object-storage-with-django)
1. [https://stackoverflow.com/questions/61101278/how-to-run-daphne-and-gunicorn-at-the-same-time](https://stackoverflow.com/questions/61101278/how-to-run-daphne-and-gunicorn-at-the-same-time)
1. [https://github.com/conda-forge/pygridgen-feedstock/issues/10](https://github.com/conda-forge/pygridgen-feedstock/issues/10)
1. [https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-20-04)