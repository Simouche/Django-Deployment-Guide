# Django-Deployment-Guide
A simple guide for a quick django app deployment on any linux vps.

# Login To Your Server Over SSH:
  this part is skipped because it doesn't belong to this context.

# Install Server Dependencies
  sudo apt-get update //to get the latest updates of the system packages.
  sudo apt-get -y upgrade // to upgrade the available upgradable packages.
  
  sudo apt-get -y install nginx // Http server, in our case it will be used as a proxy server, and assets server (css,js...)
  sudo apt-get -y install supervisor // check http://supervisord.org/ file for more details and explanations.
  
  sudo apt-get -y install python3.7 //change to the version you like, i prefer 3.7.5 (most stable according to me)
  sudo apt-get -y install virtualenv // pythoon virtual environment, will be used to isolate our project dependencies
  
  // install any database server your project require, i won't cover this as it is out of context, but in case you are using 
  MySQL (MariaDB), it requires some other dependencies to work.
  Just in case you are using MySQL database server, you should do this:
    - apt-get install python3.7-dev
    - apt-get install python3.7-dev libmysqlclient-dev
    - apt install build-essential //this is the missing requirement and it will throw so much errors if you miss it.
   
# Configure The Project Environment:
  // You should create a new system user with root privileges if you don't want to work with the root user, in my case i'm just 
  // going to use root.
  
  // change your current directory to home and create a folder to deploy your app inside:
      - cd /home
      - mkdir my_application_folder
      - cd my_application_folder
      
  // now inside your application folder, pull your code from your repository (github or private):
      - git clone https://server.address/your/repo/path
      
  // for the example the code root folder will be "application", now go inside it:
      - cd application
      
  // inside your application root folder (same level as manage.py file), we'll make the virtual environment to hold the 
     project's dependencies:
      - virtualenv --python=python3.7 venv (you can name your environment as you like, but by convention we name "venv")
      
  // now to start using the virtual environment you have to activate it:
      - source venv/bin/activate // you'll notice that the command line has changed, now it starts with "(venv)" 
      
  // to install the project dependencies, by convention all the projects have a requirements file (check the requirements.txt 
    for example), which holds the project's libraries' names and versions (optional) for the pip (package manager) to install, like so:
      - pip install -r requirements.txt 
      // if you don't have a requirements.txt file (bad practice), you'll have to go over all the libraries manually like so:
      - pip install lib_name
  
  // now you have to configure the project's settings, you'll do so by changing the settings.py file, use any editor you want,
     I personally use nano:
     - start by adding the server ip address and the host name (domain name) if provided to the allowed hosts property 
          ex: ALLOWED_HOSTS=['192.168.1.1','www.mydomainname.com','mydomainname.com']
     - configure your database settings => out of context.
     - provide a STATIC_ROOT property if not yet provided, we'll create the respective folder later, this will be used to 
          hold all the static files (js,css,images...), ex: STATIC_ROOT=os.path.join(BASE_DIR,'static_root')
     - save and exit.
     
  // change directory to the application's root (same level as manage.py) and create the static root folder:
     - mkdir static_root
     
  // once again if you are runnning MySQL database you should add this dependency (the latest stable dependency for MySQL in linux):
     - pip install mysqlclient==1.4.2.post1
     
  // makemigrations if anything changed in the model layer, and migrate:
     - python manage.py makemigration
     - python manage.py migrate
     
  // check for errors:
     - python manage.py check
     
  // check the django deployment check list (it will show some deployement requirements like https)
     - python manage.py check --deploy
     
  // test if your app configuration is ok:
     - python manage.py runserver 0.0.0.0:8001 // will start a development server on the port 8001, check from the browser if 
     your app is running ok by going to your_ip_address:80001
     
  // if everything is ok go to the next step, otherwise check if you did eveyrhing good.
  
  
# Configure Gunicorn:
  // for more details about gunicorn check https://gunicorn.org/.
  // install gunicorn in our virtual environement
      - pip install gunicorn
  // configure the gunicorn_start file to run our django app:
      - cd venv/bin
      - touch gunicorn_start
      - copy the content of gunicorn_start.txt file and make the appropriate changes according to your project
  // make the gunicorn_start file executable:
      - chmod u+x gunicorn_start
  // create the gunicorn socket folder:
      - cd ../../.. //change directory to the project folder (not the app folder) one level before manage.py 
                        inside the "my_application_folder".
      - mkdir run
 
  // that's all for the gunicorn.
  
# Configure Supervisor:
  // supervisor will be used to run your gunicorn server as a daemon (service), and keep your application up and running even
      after server crash or restart.
  // change directory to supervisor configurations folder (this folder holds all the configurations for any daemon program):
      - cd /etc/supervisor/conf.d/
      - touch anyname.conf //change the name as you want (your project name).
      - nano anyname.conf // copy the content of supervisor_config.txt and make the necessary changes according to your project.
      - cd /home/project/my_application_folder/
      - mkdir logs
      - cd logs
      - touch gunicorn-error.log
      - touch gunicorn-out.log
      
  // now you'll have to enable and start the supervisor service:
      - systemctl enable supervisor
      - systemctl start supervisor
  // now reread new scripts and update the supervisor registery:
      - supervisorctl reread
      - supervisorctl update
  // now you should see that your program name appearing and supervisor will tell you that it is added.
  // you can check your program status by:
      - sudo supervisorctl status program_name // you should see the programe name and "RUNNING" next to it with pid and uptime.
  // now your app is up and running on the gunicorn, but it can't be accessed from outside over http, that's where nginx comes.
  
  
# Configure Nginx:
  // nginx will be used as a proxy server to redirect the http requests to gunicorn server, and also serve the media and static files,
      for bigger apps you should consider making a different server to serve the media files.
  
  // create the site config file:
      - cd /etc/nginx/sites-available/
      - touch project_name (or any name you want)
      - nano project_name // copy the content of nginx_config.txt and make the necessary changes according to your project.
      - cd /home/my_application_folder/logs/ //go to the project folder and make the nginx logs files
      - touch nginx-access.log
      - touch nginx-error.log
  
  // create a symbolic link to enable the website.
      - sudo ln -s /etc/nginx/sites-available/config_file_name /etc/nginx/sites-enabled/config_file_name 
  
  // delete the nginx default website:
      - rm /etc/nginx/sites-enabled/default
  
  // restart the nginx server service:
      - service nginx restart
  // now if you go to your domain name you can access your application, but you'll notice that the static files are missing,
      no css,js...
  // that because we didn't collect the static files into the static_root folder, so:
      - cd /application
      - python manage.py collectstatic
      
  // check your website now, it should be up and running perfectly.
  // note that if you make any changes to your code, you'll have to restart the supervisor daemon only:
      - supervisorctl restart program_name
  
  // done.
