My project using AWS EC2, Python (Flask), Docker, AWS for deployment, and Jenkins for CI/CD.


---

1. Set Up Your AWS EC2 Instance

Step 1: Launch an EC2 Instance

1. Go to the AWS Management Console.


2. Launch a new EC2 Instance:

Choose Amazon Linux 2 as the operating system.

Choose an instance type like t2.micro (free tier eligible).



3. Configure Security Group:

Allow port 22 for SSH.

Allow port 80 for HTTP (web server).



4. Launch the instance and download the key pair (e.g., ec2-key.pem).



Step 2: Connect to the Instance

1. Open a terminal on your local machine.


2. Connect to the EC2 instance using SSH:

ssh -i ec2-key.pem ec2-user@<your-ec2-public-ip>



Step 3: Install Necessary Packages

Run the following commands to prepare the environment:

sudo yum update -y
sudo yum install git python3-pip docker -y
sudo pip3 install flask
sudo service docker start
sudo usermod -aG docker ec2-user  # Add your user to the docker group

Log out and back in to activate Docker permissions:

exit
ssh -i ec2-key.pem ec2-user@<your-ec2-public-ip>


---

2. Create a Simple Web Application with Flask

Step 1: Create Project Folder Structure

1. On your EC2 instance, create a directory for your project:

mkdir flask-docker-app && cd flask-docker-app


2. Create the following files:

app.py: Flask application.

requirements.txt: Python dependencies.

Dockerfile: Docker configuration.

.gitignore: Ignore unnecessary files.

Jenkinsfile: CI/CD pipeline configuration.




Step 2: Write the Flask Application

1. Open app.py for editing:

nano app.py


2. Add the following Flask code:

from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello, World! Welcome to your Dockerized Flask App!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)



Step 3: Create requirements.txt

1. Create and open requirements.txt:

nano requirements.txt


2. Add the Flask dependency:

flask



Step 4: Create .gitignore

1. Create and open .gitignore:

nano .gitignore


2. Add the following lines:

__pycache__/
*.pyc
*.pyo




---

3. Dockerize the Application

Step 1: Create the Dockerfile

1. Open Dockerfile:

nano Dockerfile


2. Add the following instructions:

FROM python:3.8-slim

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

CMD ["python", "app.py"]



Step 2: Build and Run the Docker Image

1. Build the Docker image:

docker build -t flask-docker-app .


2. Run the container to test:

docker run -d -p 80:80 flask-docker-app


3. Visit <your-ec2-public-ip> in a browser to see "Hello, World!".




---

4. Push Code to GitHub

1. Initialize Git in your project directory:

git init
git add .
git commit -m "Initial commit"


2. Link your GitHub repository:

git remote add origin <your-repo-url>
git branch -M main
git push -u origin main




---

5. Set Up Jenkins for CI/CD

Step 1: Install Jenkins

1. Install Jenkins on your EC2 instance:

sudo yum install java-11-openjdk -y
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum install jenkins -y
sudo service jenkins start


2. Open port 8080 in your EC2 security group.



Step 2: Configure Jenkins

1. Access Jenkins at http://<your-ec2-public-ip>:8080.


2. Unlock Jenkins using the password in /var/lib/jenkins/secrets/initialAdminPassword.


3. Install suggested plugins and create an admin user.



Step 3: Create a Jenkins Pipeline

1. Open Jenkins and create a new pipeline project.


2. In the pipeline, point to your Jenkinsfile in GitHub:

Repository URL: <your-repo-url>



3. Example Jenkinsfile:

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




---

6. Deploy the Application on AWS

Your Flask app is already running on port 80 of the EC2 instance. Ensure:

1. The container runs on system reboot:

docker run -d --restart always -p 80:80 flask-docker-app


2. Visit http://<your-ec2-public-ip> to access the app.




---

Final Notes

1. GitHub Repository: Push all files to GitHub.



