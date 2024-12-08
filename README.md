# Python-flask-webserver

1. Set Up Your AWS EC2 Instance
Step 1: Launch an EC2 Instance
Go to the AWS Management Console.
Launch a new EC2 Instance:
Choose Amazon Linux 2 as the operating system.
Choose an instance type like t2.micro (free tier eligible).
Configure Security Group:
Allow port 22 for SSH.
Allow port 80 for HTTP (web server).
Launch the instance and download the key pair (e.g., ec2-key.pem).
Step 2: Connect to the Instance
Open a terminal on your local machine.
Connect to the EC2 instance using SSH:
bash
Copy code
ssh -i ec2-key.pem ec2-user@<your-ec2-public-ip>
Step 3: Install Necessary Packages
Run the following commands to prepare the environment:

bash
Copy code
sudo yum update -y
sudo yum install git python3-pip docker -y
sudo pip3 install flask
sudo service docker start
sudo usermod -aG docker ec2-user  # Add your user to the docker group
Log out and back in to activate Docker permissions:

bash
Copy code
exit
ssh -i ec2-key.pem ec2-user@<your-ec2-public-ip>
2. Create a Simple Web Application with Flask
Step 1: Create Project Folder Structure
On your EC2 instance, create a directory for your project:
bash
Copy code
mkdir flask-docker-app && cd flask-docker-app
Create the following files:
app.py: Flask application.
requirements.txt: Python dependencies.
Dockerfile: Docker configuration.
.gitignore: Ignore unnecessary files.
Jenkinsfile: CI/CD pipeline configuration.
Step 2: Write the Flask Application
Open app.py for editing:
bash
Copy code
nano app.py
Add the following Flask code:
python
Copy code
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello, World! Welcome to your Dockerized Flask App!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)
Step 3: Create requirements.txt
Create and open requirements.txt:
bash
Copy code
nano requirements.txt
Add the Flask dependency:
Copy code
flask
Step 4: Create .gitignore
Create and open .gitignore:
bash
Copy code
nano .gitignore
Add the following lines:
markdown
Copy code
__pycache__/
*.pyc
*.pyo
3. Dockerize the Application
Step 1: Create the Dockerfile
Open Dockerfile:
bash
Copy code
nano Dockerfile
Add the following instructions:
dockerfile
Copy code
FROM python:3.8-slim

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

CMD ["python", "app.py"]
Step 2: Build and Run the Docker Image
Build the Docker image:
bash
Copy code
docker build -t flask-docker-app .
Run the container to test:
bash
Copy code
docker run -d -p 80:80 flask-docker-app
Visit <your-ec2-public-ip> in a browser to see "Hello, World!".
4. Push Code to GitHub
Initialize Git in your project directory:
bash
Copy code
git init
git add .
git commit -m "Initial commit"
Link your GitHub repository:
bash
Copy code
git remote add origin <your-repo-url>
git branch -M main
git push -u origin main
5. Set Up Jenkins for CI/CD
Step 1: Install Jenkins
Install Jenkins on your EC2 instance:
bash
Copy code
sudo yum install java-11-openjdk -y
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum install jenkins -y
sudo service jenkins start
Open port 8080 in your EC2 security group.
Step 2: Configure Jenkins
Access Jenkins at http://<your-ec2-public-ip>:8080.
Unlock Jenkins using the password in /var/lib/jenkins/secrets/initialAdminPassword.
Install suggested plugins and create an admin user.
Step 3: Create a Jenkins Pipeline
Open Jenkins and create a new pipeline project.
In the pipeline, point to your Jenkinsfile in GitHub:
Repository URL: <your-repo-url>
Example Jenkinsfile:
groovy
Copy code
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'docker build -t flask-docker-app .'
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker run -d -p 80:80 flask-docker-app'
            }
        }
    }
}
6. Deploy the Application on AWS
Your Flask app is already running on port 80 of the EC2 instance. Ensure:

The container runs on system reboot:
bash
Copy code
docker run -d --restart always -p 80:80 flask-docker-app
Visit http://<your-ec2-public-ip> to access the app.
