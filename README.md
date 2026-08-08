------------------------------------------------------------------------
PROJECT STRUCTURE
------------------------------------------------------------------------
- app.py            : Simple Python Flask Web Server
- requirements.txt  : Dependencies (Flask)
- Dockerfile        : Container recipe
- Jenkinsfile       : Pipeline definition (optional)
- README.txt        : This documentation file
------------------------------------------------------------------------
SETUP & RUNNING INSTRUCTIONS
------------------------------------------------------------------------
1. Run Jenkins with Docker Access:
   Start your Jenkins container by mounting the host's Docker socket:
   
   docker run -d \
     -p 8080:8080 \
     -p 50000:50000 \
     -v /var/run/docker.sock:/var/run/docker.sock \
     -v jenkins_home:/var/jenkins_home \
     --name jenkins \
     jenkins/jenkins:lts
2. Install Docker CLI inside Jenkins:
   Enter the running Jenkins container as root and install the Docker CLI:
   
   docker exec -it --user root jenkins bash
   apt-get update && apt-get install -y docker.io
   exit
3. Create Jenkins Freestyle Job:
   - Create a new Freestyle project in Jenkins named "flask-app-freestyle".
   - Set SCM to "Git" and use your repository URL.
   - Add an "Execute shell" build step and paste the deployment script.
   - Click "Build Now" to run.
========================================================================
