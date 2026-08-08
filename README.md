# Jenkins CI/CD: Dockerized Flask Application 
This guide provides a step-by-step setup to build, push, and deploy a Python Flask application using **Jenkins** and **Docker-outside-of-Docker (DooD)**.
---
## 🏗️ Step 1: Set Up Jenkins Container (DooD)
To let Jenkins talk to your host's Docker daemon, you must mount the Docker socket file when starting Jenkins.
### Command to Run Jenkins:
```bash
docker run -d --name jenkins --restart unless-stopped \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e TZ=Asia/Kolkata \
  jenkins/jenkins:lts
💡 What this command means:
docker run -d: Runs the container in the background (detached mode).
--name jenkins: Names the container jenkins so you can easily reference it.
--restart unless-stopped: Automatically restarts Jenkins if the system reboots.
-p 8080:8080: Maps Jenkins web UI to your local browser port 8080.
-v jenkins_home:/var/jenkins_home: Saves your Jenkins settings, users, and pipelines on your computer so they are not deleted if the container stops.
-v /var/run/docker.sock:/var/run/docker.sock: (Crucial) Connects the Jenkins container to the host's Docker daemon.
jenkins/jenkins:lts: Downloads and uses the official Long Term Support version of Jenkins.
🔑 Step 2: Grant Permissions to the Docker Socket
By default, the Docker socket is locked for security. You must unlock it so Jenkins can use it.

Command to Unlock Socket:
bash


docker exec -u root -it jenkins chmod 666 /var/run/docker.sock
💡 What this command means:
docker exec: Runs a command inside a container that is already running.
-u root: Runs the command as the Administrator (root) user.
-it jenkins: Opens an interactive session inside the jenkins container.
chmod 666 /var/run/docker.sock: Changes the socket permissions so anyone inside the container can read and write to it.
🐳 Step 3: Install Docker Client inside Jenkins
Jenkins needs the docker command utility installed inside it so it can build images.

Command to Install Docker:
bash


docker exec -u root -it jenkins bash -lc \
  'curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh'
💡 What this command means:
curl -fsSL https://get.docker.com -o get-docker.sh: Downloads the official Docker installation script.
sh get-docker.sh: Executes the script to install the Docker CLI inside the Jenkins container.
🛠️ Step 4: Configure Jenkins Credentials
To push images to Docker Hub securely, add your credentials:

Open Jenkins: http://localhost:8080
Go to: Manage Jenkins 
→
→ Credentials 
→
→ System 
→
→ Global credentials (unrestricted).
Click Add Credentials:
Kind: Username with password
Username: your_dockerhub_username
Password: your_dockerhub_password
ID: docker-hub-creds
Click Create.
⚙️ Step 5: Jenkins Execute Shell Script
In your Freestyle job, add an Execute Shell build step and paste this script:

bash


# 1. Set your credentials directly
DOCKER_USER="your_dockerhub_username"
DOCKER_PASS="your_dockerhub_password"
# 2. Check if files are inside a subfolder (like python/) and navigate to it
if [ -f "Dockerfile" ]; then
    echo "Dockerfile found in the root directory."
elif [ -f "python/Dockerfile" ]; then
    echo "Dockerfile found in python/ folder. Navigating..."
    cd "python"
else
    echo "ERROR: Dockerfile not found!"
    exit 1
fi
# 3. Build the Docker Image
# ($BUILD_NUMBER is a variable Jenkins updates automatically for every run)
docker build -t $DOCKER_USER/flask-doc:$BUILD_NUMBER .
docker tag $DOCKER_USER/flask-doc:$BUILD_NUMBER $DOCKER_USER/flask-doc:latest
# 4. Login and Push to Docker Hub
echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
docker push $DOCKER_USER/flask-doc:$BUILD_NUMBER
docker push $DOCKER_USER/flask-doc:latest
docker logout
# 5. Deploy
# Stops and deletes the old container to free up port 5000, then starts the new one
docker stop cwvj-flask || true
docker rm cwvj-flask || true
docker run -d --name cwvj-flask -p 5000:5000 $DOCKER_USER/flask-doc:latest
