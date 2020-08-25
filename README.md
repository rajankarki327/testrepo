### APIs for Project Management System

# Instructions to deploy the application on a server

1. Log in the desired server and navigate to the location where we we want our project to be setup. 
2. Make a directory 
    
        mkdir pms_api_project
3. Create and activate a virtual environment for the project
    
       virtualenv --python=[Path to python3.6 on server] [name of vitrualenv] 
         
    -- To activate it
        
        source venv/bin/activate
4. Clone the repository
     
        git clone git@repo.ekbana.info:data_analysis_platform/pms_api.git
                 
5. Navigate into the repository and install dependencies
    >pip3 install -r requirements.txt
    --also install gunicorn and psycopg2
    >pip3 install gunicorn pyscopg2
6. Make the env file as per env.example and test if the project runs
    >python3 manage.py runserver




# Configuring Gunicorn
1. We need to a socket file and a service file for each project

1.1. Navigate to /etc/systemd/system/ and make the socket file
  
  >sudo nano pms_api.socket

1.2.Paste in and save (Change the project names and path accordingly)
    
    [Unit]
    Description=gunicorn socket
    
    [Socket]
    ListenStream=/path/to/project/pms_api.sock
    
    [Install]
    WantedBy=sockets.target   


1.3 Make the service file
    
    >sudo nano pms_api.service
   Paste in and save (Change the filenames and path accordingly and 
   use the name of directory in place of 'directory' that contains the wsgi file )
    
    [Unit]
    Description=gunicorn daemon
    Requires=pms_api.socket
    After=network.target
    
    [Service]
    User=saque
    Group=www-data
    WorkingDirectory=/path/to/project
    ExecStart=/path/to/project/venv/bin/gunicorn --workers 3 --bind unix:/path/to/project/pms_api.sock directory.wsgi:application
    
    [Install]
    WantedBy=multi-user.target
    
    
2. Enable and start the gunicorn service we just created
    
       sudo systemctl start pms_api
       sudo systemctl enable pms_api
       
    This should create the pms_api.sock file in project root directory




# Configuring Nginx
1. Navigate to /etc/nginx/sites-available/
2. Make file with projectname 
    
        sudo nano pms_api
3. Paste and save (Change the port number and path accordingly)
    
        server {
        listen **port_number**;
        server_name **server_domain_or_IP**;
    
        location = /favicon.ico { access_log off; log_not_found off; }
        location /static/ {
            root /path/to/project;
        }
    
        location / {
            include proxy_params;
            proxy_pass http://unix:/path/to/project/pms_api.sock;
        }
        }

4. Linking it to the sites-enabled directory
    
        sudo ln -s /etc/nginx/sites-available/pms_api /etc/nginx/sites-enabled

5. Test the configuration and restart nginx if config is ok

        sudo nginx -t
        sudo systemctl restart nginx

The project should be running by now unless creating and creating the database is required
Follow the link below for steps to create a database in postgres
 [https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-16-04]

There might be need to migrate the models if a new database was created
    
    python3 manange.py migrate
    
    
    
    
    
# Pulling new changes in the project
1. Navigate to project repo, activate venv if needed and pull the changes from the desired branch
2. Restart the Gunicorn service for this project
    
        sudo service pms_api restart
    
    
    
    
    
                                                           
                   
                         
