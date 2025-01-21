Hosting a Flask Application on EC2 with Gunicorn and Nginx

The following steps were followed to set up a Flask app on an EC2 instance, using Gunicorn as the WSGI server and Nginx as a reverse proxy:

    Install Python Virtualenv
    Updated the package list and installed Python virtualenv:

sudo apt-get update
sudo apt-get install python3-venv

Set up the Virtual Environment
Created a project folder, set up a virtual environment, and activated it:

mkdir project
cd project
python3 -m venv venv
source venv/bin/activate

Install Flask
Installed Flask in the virtual environment:

pip install flask

Clone the Flask Application
Cloned the Flask app from the GitHub repository:

git clone <link>

Ran the app to ensure it worked:

python run.py

Install Gunicorn
Installed Gunicorn and ran it to serve the Flask app:

pip install gunicorn
gunicorn -b 0.0.0.0:8000 app:app

Use systemd to Manage Gunicorn
Created a systemd unit file to manage Gunicorn as a service:

sudo nano /etc/systemd/system/project.service

Added the following configuration:

[Unit]
Description=Gunicorn instance for a de-project
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/project
ExecStart=/home/ubuntu/project/venv/bin/gunicorn -b localhost:8000 app:app
Restart=always

[Install]
WantedBy=multi-user.target

Enabled and started the Gunicorn service:

sudo systemctl daemon-reload
sudo systemctl start project
sudo systemctl enable project

Run Nginx Webserver
Installed and started Nginx:

sudo apt-get install nginx
sudo systemctl start nginx
sudo systemctl enable nginx

Configured Nginx to proxy requests to Gunicorn by editing the default config:

sudo nano /etc/nginx/sites-available/default

Added the following configuration:

upstream flask_project {
    server 127.0.0.1:8000;
}

location / {
    proxy_pass http://flask_project;
}

Restarted Nginx to apply changes:

    sudo systemctl restart nginx

    Verified the app was accessible through Nginx by visiting the EC2 public IP.

Create Lambda Layers

To manage dependencies for the Lambda function, the following steps were followed:

    Create a Directory for the Layer
    Created a directory for the layer:

mkdir lambda_layer
cd lambda_layer

Install Dependencies
Installed necessary libraries (e.g., pandas, boto3) in the directory:

pip install pandas -t .
pip install boto3 -t .

Remove Unnecessary Files
Removed unnecessary files to reduce the layer size.

Create a Zip File
Created a zip file containing the contents of the directory:

zip -r lambda_layer.zip .

Upload as a Layer
Uploaded the zip file to AWS Lambda as a new layer.

Attach the Layer to the Lambda Function
Attached the layer to the Lambda function in the AWS Lambda console.

Test the Lambda Function
Tested the Lambda function to ensure it worked with the new layer.
