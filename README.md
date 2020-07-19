# Django-Deployment-Guide
A simple guide for a quick django app deployment on any linux vps.

# Steps:
## Login To Your Server Over SSH:
  This part is skipped because it doesn't belong to this context.

## Install Server Dependencies

* Get the latest updates of the system packages.

```console
foo@bar:~$ sudo apt-get update
```

* Upgrade the available upgradable packages.

```console
foo@bar:~$ sudo apt-get -y upgrade
```

* Install nginx (proxy server, and assets server (css,js...))

```console
foo@bar:~$ sudo apt-get -y install nginx
```

* Install supervisor (http://supervisord.org/)

```console
foo@bar:~$ sudo apt-get -y install supervisor
```

* Install Python3.7

```console
foo@bar:~$ sudo apt-get -y install python3.7
```

* Install pythoon virtual environment (will be used to isolate our project dependencies)

```console
foo@bar:~$ sudo apt-get -y install virtualen
```

Install any database server your project require, i won't cover this as it is out of context. 

In case you are using MySQL (MariaDB), it requires some other dependencies to work.

```console
foo@bar:~$ apt-get install python3.7-dev
```
```console
foo@bar:~$ apt-get install python3.7-dev libmysqlclient-dev
```
```console
foo@bar:~$ apt install build-essential
```
   
## Configure The Project Environment:

* You should create a new system user with root privileges if you don't want to work with the root user, in my case i'm just going to use root.
  
* change your current directory to home and create a folder to deploy your app inside:

```console
foo@bar:~$ cd /home
```

```console
foo@bar:~$/home/ mkdir my_application_folder
```

```console
foo@bar:~$/home/ cd my_application_folder
```

* now inside your application folder, pull your code from your repository (github or private):

```console
foo@bar:~$/home/my_application_folder git clone https://server.address/your/repo/path
```

* for the example the code root folder will be "application", now go inside it:

```console
foo@bar:~$/home/my_application_folder cd application
```
      
* inside your application root folder (same level as manage.py file), we'll make the virtual environment to hold the project's dependencies:

```
application/
└── manager.py
```
```console
foo@bar:~$/home/my_application_folder/application/ virtualenv --python=python3.7 venv
```
(you can name your environment as you like, but by convention we name "venv")
      
* Start using the virtual environment you have to activate it:

```console
foo@bar:~$/home/my_application_folder/application/ source venv/bin/activate
```
you'll notice that the command line has changed, now it starts with "(venv)" 
      
* install the project dependencies, by convention all the projects have a requirements file (check the requirements.txt for example), which holds the project's libraries' names and versions (optional) for the pip (package manager) to install, like so:
  
```console
foo@bar:~$/home/my_application_folder/application/ pip install -r requirements.txt 
```

If you don't have a requirements.txt file (bad practice), you'll have to go over all the libraries manually like so:

```console
foo@bar:~$/home/my_application_folder/application/ pip install lib_name 
```
  
* now you have to configure the project's settings, you'll do so by changing the settings.py file, use any editor you want,I personally use nano:
  
     * start by adding the server ip address and the host name (domain name) if provided to the allowed hosts property 
          ex: `ALLOWED_HOSTS=['192.168.1.1','www.mydomainname.com','mydomainname.com']`
     * configure your database settings => out of context.
     * provide a STATIC_ROOT property if not yet provided, we'll create the respective folder later, this will be used to hold all the static files (js,css,images...), ex: STATIC_ROOT=os.path.join(BASE_DIR,'static_root')
     * save and exit.
     
* change directory to the application's root (same level as manage.py) and create the static root folder:

```console
foo@bar:~$/home/my_application_folder/application/ mkdir static_root
```
     
* once again if you are runnning MySQL database you should add this dependency (the latest stable dependency for MySQL in linux):

```console
foo@bar:~$/home/my_application_folder/application/ pip install mysqlclient==1.4.2.post1
```  

* makemigrations if anything changed in the model layer, and migrate:

```console
foo@bar:~$/home/my_application_folder/application/ python manage.py makemigration
```   

```console
foo@bar:~$/home/my_application_folder/application/ python manage.py migrate
```   
     
* check for errors:

```console
foo@bar:~$/home/my_application_folder/application/ python manage.py check
```  
     
* check the django deployment check list (it will show some deployement requirements like https)

```console
foo@bar:~$/home/my_application_folder/application/ python manage.py check --deploy
```       
* test if your app configuration is ok:

```console
foo@bar:~$/home/my_application_folder/application/ python manage.py runserver 0.0.0.0:8001
```

Your app is running ok by going to your_ip_address:80001
     
if everything is ok go to the next step, otherwise check if you did eveyrhing good.
   
## Configure Gunicorn:
**for more details about gunicorn check https://gunicorn.org/**

* install gunicorn in our virtual environement
```console
foo@bar:~$/home/my_application_folder/application/ pip install gunicorn
```

* configure the gunicorn_start file to run our django app:

```console
foo@bar:~$/home/my_application_folder/application/ cd venv/bin
```
```console
foo@bar:~$/home/my_application_folder/application/ touch gunicorn_start
```

copy the content of gunicorn_start.txt file and make the appropriate changes according to your project

* make the gunicorn_start file executable

```console
foo@bar:~$/home/my_application_folder/application/ chmod u+x gunicorn_start
```

* create the gunicorn socket folder:

```console
foo@bar:~$/home/my_application_folder/application/ cd ../../
```

change directory to the project folder (not the app folder) one **level before manage.py**  inside the "my_application_folder".

```console
foo@bar:~$/home/my_application_folder/ mkdir run
```

that's all for the gunicorn.
  
## Configure Supervisor:

supervisor will be used to run your gunicorn server as a daemon (service), and keep your application up and running even after server crash or restart.

* change directory to supervisor configurations folder (this folder holds all the configurations for any daemon program):

```console
foo@bar:~$/home/my_application_folder/ cd /etc/supervisor/conf.d/
```

```console
foo@bar:~$/home/my_application_folder/etc/supervisor/conf.d/ touch anyname.conf
```
change the **anyname** as you want (your project name).

```console
foo@bar:~$/home/my_application_folder/etc/supervisor/conf.d/ nano anyname.conf
```
copy the content of supervisor_config.txt and make the necessary changes according to your project.

```console
foo@bar:~$/home/my_application_folder/etc/supervisor/conf.d/ cd /home/project my_application_folder/
```
```console
foo@bar:~$/home/my_application_folder/ mkdir logs
```
```console
foo@bar:~$/home/my_application_folder/ cd logs
```
```console
foo@bar:~$/home/my_application_folder/logs touch gunicorn-error.log
```
```console
foo@bar:~$/home/my_application_folder/logs touch gunicorn-out.log
```

      
* now you'll have to enable and start the supervisor service:
  
```console
foo@bar:~$/home/my_application_folder/logs systemctl enable supervisor
```

```console
foo@bar:~$/home/my_application_folder/logs systemctl start supervisor
```

* now reread new scripts and update the supervisor registery:

```console
foo@bar:~$/home/my_application_folder/logs supervisorctl reread
```
```console
foo@bar:~$/home/my_application_folder/logs supervisorctl update
```
* now you should see that your program name appearing and supervisor will tell you that it is added.

* you can check your program status by:
```console
foo@bar:~/home/my_application_folder/logs$ sudo supervisorctl status program_name
```

you should see the programe name and **"RUNNING"** next to it with pid and uptime.

now your app is up and running on the gunicorn, but it can't be accessed from outside over http, that's where nginx comes.
  
  
## Configure Nginx:
nginx will be used as a proxy server to redirect the http requests to gunicorn server, and also serve the media and static files,
      for bigger apps you should consider making a different server to serve the media files.
  
   * create the site config file:
   ```console
   foo@bar:~$ cd /etc/nginx/sites-available/
   foo@bar:~/etc/nginx/sites-available/$ touch project_name (or any name you want)
   foo@bar:~/etc/nginx/sites-available/$ nano project_name (copy the content of nginx_config.txt and make the necessary changes according to your project)
   foo@bar:~/etc/nginx/sites-available/$ cd /home/my_application_folder/logs/ (go to the project folder and make the nginx logs files)
   foo@bar:~/home/my_application_folder/logs/$ touch nginx-access.log
   foo@bar:~/home/my_application_folder/logs/$ touch nginx-error.log
   ```
  * create a symbolic link to enable the website.
  ```console
      foo@bar:~/home/my_application_folder/logs/$ sudo ln -s /etc/nginx/sites-available/config_file_name /etc/nginx/sites-enabled/config_file_name 
  ```
  * delete the nginx default website:
  ```console
      foo@bar:~/home/my_application_folder/logs/$ rm /etc/nginx/sites-enabled/default
  ```  
  * restart the nginx server service:
  ```console
      foo@bar:~/home/my_application_folder/logs/$ service nginx restart
  ```  
  now if you go to your domain name you can access your application, but you'll notice that the static files are missing,no css,js...
  *that because we didn't collect the static files into the static_root folder, so:
      * change directory to your application, same level as manage.py
      ```console
      foo@bar:~/path/to/app/$ python manage.py collectstatic
      ```       
  check your website now, it should be up and running perfectly.
  note that if you make any changes to your code, you'll have to restart the supervisor daemon only:
  ```console
      foo@bar:~$ supervisorctl restart program_name
  ```    
 # Done.
