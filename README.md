Prerequisites:
1. Nexus 
2. Sonarqube
3. Maven Installed

------------------------------------------------------------------------------------------------------------------------------------------------
1. Application Overview
2. Git use cases
3. Jenkins Job Creation 
4. Maven UNIT TEST
5. Maven Integration TEST
6. Maven Building jar/War files
7. SonarQube Configuration
8. Sonarqube-webhook Configuration 
9. Static code Analysis 
10. QualityGate Status  
11. Nexus Repo Overview
12. Release Repo creation 
13. Snapshot repo creation  & Configuration
14. Error Debugging
15. Multistage DockerFile 
16. Docker Image Build
17. Docker Image Push
18. To be continued...
For jenkins installation:

INSTALLATIONS ...
create 3 instances ubuntu 22.04 for jenkins,sonarqube & nexus
initiate git in local and push code files to github
   git init
   git config --global user.name sahunirmal
   git config --global user.email nirmalendusahu@gmail.com
   git add .
   git commit -m "source-code file added"
   git remote add origin https://github.com/sahunirmal/demo-counter-app.git
   git branch
   git push -u origin master
Connect to ubuntu server
   ssh -i c:\Users\nirma\Downloads\19sep.pem ubuntu@ec2-44-223-109-206.compute-1.amazonaws.com
   sudo apt-get update
vi jenkins.sh  and enter  bellow scripts
   sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
   https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
   echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
   https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
   /etc/apt/sources.list.d/jenkins.list > /dev/null
   sudo apt-get update
   sudo apt-get install jenkins -y
save file and give permission to jenkins.sh
   sudo chmod +x jenkins.sh
run scripts
   ./jenkins.sh
install docker also using scripts
   vi docker.sh    and enter below commands

   # Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

save file , give execute permissinon 
   sudo chmod +x docker.sh
   /.docker.sh
give permission to ubuntu user to run docker commands
   sudo chmod 666 /var/run/docker.sock
   or,
   sudo usermod -aG docker ubuntu
Now setting up sonarqube server using docker container. install docker in this vm using above docker.sh script and give permission to everyuser to run docker command.
   sudo chmod 666 /var/run/docker.sock
download sonar container 
   docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

   
   
   

