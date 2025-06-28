# Deploying django project to vps:

### Tutorial video
[![Django ubuntu deployment](https://markdown-videos-api.jorgenkh.no/url?url=https%3A%2F%2Fyoutu.be%2FRsrJzKPigc4)](https://youtu.be/RsrJzKPigc4)

- Prerequisites: 
    - Create a requirements.txt file that contains all packages used in project:  
        - `pip freeze > requirements.txt`
    - Set up static files:
        - Install whitenoise:
            - `pip install whitenoise`
            - `pip freeze > requirements.txt`
        - Paste the following at the top of middleware list:
            - `"whitenoise.middleware.WhiteNoiseMiddleware",`
        - Add the following:
            ```python
            STATIC_URL = '/static/'
            STATIC_ROOT = BASE_DIR / "staticfiles"
            STATICFILES_DIRS = [
                os.path.join(BASE_DIR, 'static'),
            ]

            STATICFILES_STORAGE = "whitenoise.storage.CompressedManifestStaticFilesStorage"
            ```
        - Collect static files:
            - `python manage.py collectstatic`

    - Push to github repository

1. Ssh into server:
    - run:
        - `ssh username@server_ip_address`
        - Enter password
        &nbsp;

2. Update & Upgrade Server:
    - `sudo apt-get update`
    - `sudo apt-get upgrade`
    &nbsp;

3. Install python & pip:
    - `sudo apt install python3`
    - `sudo apt install python3-pip`
    &nbsp;

4. Create & activate virtual environment:
    - `virtualenv /opt/myproject`
    - `source /opt/myproject/bin/activate`
    &nbsp;

5. Clone Repository:
    - `cd /opt/myproject`
    - `mkdir myproject`
        - It may seem redundant to have two directories with the same name; however, it makes it so that your virtualenv name and project name are the same.
    - `cd myproject`
    - check if git is installed:
        - `git status`
        - `sudo apt-get install git`
    - Clone repo:
        `git clone repo-url`
    - install requirements:
        - `cd repo-name`
        - `pip install -r requirements.txt`
        - `pip install gunicorn`
    &nbsp;

6. Nginx Configuration:
    - Install nginx:
        - `sudo apt install nginx`
    - `sudo nano /etc/nginx/sites-available/myproject`
    - Type the following into the file:
         ```nginx
        server {
            listen 80;
            server_name yourip;

            access_log /var/log/nginx/website-name.log;

            location /static/ {
                alias /opt/myproject/myproject/path-to-static-files/;
            }

            location / {
                proxy_pass http://127.0.0.1:8000;
                proxy_set_header X-Forwarded-Host $server_name;
                proxy_set_header X-Real-IP $remote_addr;
                add_header P3P 'CP="ALLDSP COR PSAa PSDa OURNOR ONL UNI COM NAV"';
            }
        }
         ```
    - Now we need to set up a symbolic link in the /etc/nginx/sites-enabled directory that points to this configuration file. That is how NGINX knows this site is active. Change directories to /etc/nginx/sites-enabled like this:
        - `cd /etc/nginx/sites-enabled`
        - `sudo ln -s ../sites-available/myproject`

    - Go to:
        - `/etc/nginx/nginx.conf`
        - uncomment this line:
            - `# server_names_hash_bucket_size 64;`

    - Restart nginx:
        - `sudo service nginx restart`
    
    &nbsp;

7. Adjusting the Firewall

    - Before testing Nginx, the firewall software needs to be adjusted to allow access to the service. Nginx registers itself as a service with ufw upon installation, making it straightforward to allow Nginx access.

    - `sudo apt-get install ufw`
    - `sudo ufw allow 8000`

    - Restart nginx:
        - `sudo service nginx restart`
    &nbsp;

8. Testing configuration:

    - Run gunicorn:
        - `cd /opt/myproject/myproject/repo-name`
        - `gunicorn --bind 0.0.0.0:8000 project_name.wsgi`
    - Visit your project on your server ip address on port 8000:
        - `yourServerIp:8000`

    &nbsp;

9. Connecting domain:
    - open your domain registrar and open your dns settings for the specified domain

    - Add an A record with name @, pointing to the ip address of the server you are hosting your project on

    - Save changes

    - Open your nginx configuration file:
        - `sudo nano /etc/nginx/sites-available/myproject`

    - Make the following changes:

        ```nginx
        server {
            listen 80;
            server_name yourdomain.com www.yourdomain.com;

            access_log /var/log/nginx/website-name.log;

            location /static/ {
                alias /opt/myproject/myproject/path-to-static-files/;
            }

            location / {
                proxy_pass yourServerIp:8000;
                proxy_set_header X-Forwarded-Host $server_name;
                proxy_set_header X-Real-IP $remote_addr;
                add_header P3P 'CP="ALL DSP COR PSAa PSDa OUR NOR ONL UNI COM NAV"';
            }
        }
        ```
    - Restart nginx:
        - `sudo service nginx restart`
        
    - Wait for dns changes to propagate and check your website on your connected domain
    &nbsp;

10. Install SSL on vps with Let's Encrypt (Requires domain):

    - `sudo apt install certbot`
    - `sudo apt install certbot python3-certbot-nginx`
    - run the following and follow the process:
        - `sudo certbot --nginx -d your_domain.com`
    - check nginx configuration:
        - `sudo nginx -t`
    - reload nginx:
        `sudo systemctl reload nginx`
    - run gunicorn:
        - `gunicorn --bind 0.0.0.0:8000 project_name.wsgi`

- Run gunicorn in the background:
    `nohup gunicorn --bind 0.0.0.0:8000 project_name.wsgi &`
    &nbsp;

- kill gunicorn running in background:
    - `pkill gunicorn`
    &nbsp;

- Making changes

# Refrences

## Articles:
 - https://www.digitalocean.com/community/tutorials/how-to-deploy-a-local-django-app-to-a-vps

- https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-22-04

- https://stackoverflow.com/questions/37339383/nginx-gunicorn-django-not-working
