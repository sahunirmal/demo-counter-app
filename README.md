## Prerequisites:
1. Nexus 
2. Sonarqube
3. Maven Installed

# Scenario Covered
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
 ```
   git init
   git config --global user.name sahunirmal
   git config --global user.email nirmalendusahu@gmail.com
   git add .
   git commit -m "source-code file added"
   git remote add origin https://github.com/sahunirmal/demo-counter-app.git
   git branch
   git push -u origin master
   ```
For connecting to private repo through https: create Personal Access Token in github's setting and paste in cli after  adding remote origin in password section.
or, For SSH connction execute `ssh-keygen` in CLI . Go to github's setting SSH and GPG key section. Create new and paste the generated key.

## 1.Connect to 1st ubuntu server and install jenkins

  ```
ssh -i c:\Users\nirma\Downloads\19sep.pem ubuntu@public_ip
  ```
  ```
 sudo apt-get update
  ```
Install java 
```
sudo apt install openjdk-17-jdk
```
vi jenkins.sh  and enter  bellow scripts
  ```
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
   https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
   echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
   https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
   /etc/apt/sources.list.d/jenkins.list > /dev/null
   sudo apt-get update
   sudo apt-get install jenkins -y
   sudo systemctl enable jenkins
   sudo systemctl start jenkins
```
save file and give permission to jenkins.sh
  ```
  sudo chmod +x jenkins.sh
```
run scripts
   `./jenkins.sh`
   
## 2.Setting up sonar  in 2nd server using docker container
   vi docker.sh    and enter below commands
```
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
```
save file , give execute permissinon and run script
```
sudo chmod +x docker.sh
   /.docker.sh
```
give permission to ubuntu user to run docker commands
   `sudo chmod 666 /var/run/docker.sock`
   or,
   `sudo usermod -aG docker ubuntu`
download and run sonar container 
  ```
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
  ```
In case container stops or rebooting ec2 execute below commands
```
 systemctl start docker
 docker start sonar
```
connect to sonar ip:9000 through browser. Make sure port 9000 is open in security group.   

## 3. Setting up nexus server using docker container

install docker using above scripts docker.sh . and give permission to user to run docker commands
    ```
    sudo chmod 666 /var/run/docker.sock
    ```
run nexus container and open port 8081 in vm's sg.
    ```
    docker run -d --name nexus3 -p 8081:8081 sonatype/nexus3
    ```
Access ip-of-vm:8081, sign in to nexus3 . user-name admin . get password form below steps
   
   ```
    docker exec -it nexus3 /bin/bash
    cd sonatype-work/nexus3/
    cat admin.password
   ```
Paste passwd select option "Enable anonymous access"

To Install Nexus Repository Manager directly on Ubuntu 22.04 VM, visit:  [https://www.howtoforge.com/how-to-install-nexus-repository-manager-on-ubuntu-22-04/](url)

## Setting up jenkins server

install plugin > SonarQube Scanner , SonarQube Generic Coverage, Sonar Gerrit, Quality Gates, Sonar Quality Gates
Goto manage jenkins> configure system> add tool maven as "mymaven"
Add sonar server 

In Jenkins, go to Manage Jenkins -> Manage Credentials -> (global) -> Add Credentials.
Add your GitHub credentials (username and personal access token)
create pipeline job -> configuration > pipeline > definition > pipepline script from SCM > give git ,repo url and credentials and script path "jenkinsfile" ,branch master or main > save

### Configure Sonal plugins
manage jenkins > configure System > SonarQube Servers >installations > give name >server url > create Server authentication token(kind= Secret text,get secret from sonarqube server > adminsitration >security> users>Token of Adminstrator>generaate Token > copy and paste in secret section in jenkins >give ID and Desription as sonar-api > ADD ) > select "sonar-api" token >Apply and Save.

### SonarQube Analysis stage-
Generate script for using pipeline syntax generator > select withSonarQubeEnv plugin in step section > select "sonar-api" key that just created .

### Quality Gate status stage-
Generate script for using pipeline syntax generator > select waitForQualityGate  plugin in step section > serever authentication token "sonar-api" 
While running this pipeline stage we will get into a loop "status in Pending" . To avoid the "status in Pending" loop and establish a two-way handshake between Jenkins and SonarQube, you need to ensure that Jenkins can wait for SonarQube's quality gate result. This requires proper configuration of the SonarQube WEBHOOK to notify Jenkins of the analysis completion and quality gate status.
Goto sonarqube server> Administration > Configuration > Webhhoks > create > give name and URL as http://jenkinsserverip:8080/sonarqube-webhook/  

### Sign in to nexus repo > go to setting > Repository > create  2 repo Release and SNAPSHOT repo of type maven2 (hosted).
To connect this nexus repo to jenkins to upaload Artifect after building > download  Nexus Artifect Uploader PLUGIN
Generate pipeline syntax > select from steps dropdown >nexusArtifactUploader: Nexus Artifact Uploader >NEXUS3 > add credentials (username and password of nexus,)ID and Description) > Groupid, version, nexusrepository name that created just before >Add artifact id ,type jar,file(target/Uber.jar ) from pom.xml file. >generate code and put in jenkins pipeline code. Run pipeline > Uber.jar file will be pushed to Nexux repository.

### To fetch/extrat version from pom.xml dynamically 
Download Pipeline Utility Steps plugin > define a variable which will read pom.xml i.e 
def readPomVersion = readMavenPom file: 'pom.xml'    call it in script
version: "${readPomVersion.version}"
Write a condition which will push packaged file to nexus released/SNAPSHOT repo
def nexusRepo = readPomVersion.version.endswith("SNAPSHOT") ? "demoapp-snapshot" : "demoapp-release"      call it in script
repository: nexusRepo

### Next create multistage docker file
docker should be installed in jenkins server to run docker commands
add docker hub credentials using Withcredentials: Bind credentials to variables > Bindings "secret text"  > give dockerhub password > type username and password >generate syntax and paste in pipeline

Project link:
 * Part1 [https://www.youtube.com/watch?v=Yk7k3yEguQA](url) 
 * PArt2 [https://www.youtube.com/watch?v=MpRd_nJEx_8&t=1173s](url) <br>
Install AWS CLI [https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html](url) <br>
You have terraform module to create EKS cluster. Execute
```
teraform init
terraform plan --var-file=./config/terraform.tfvars
terraform apply --var-file=./config/terraform.tfvars --auto-approve
```
### Setup to connect to eks cluster form Jenkins server
- Goto jenkins dashboard > manage jenkins > credentials > System > Add credentials > kind (Secret text) > add  Access_key_id in secret section > ID and Descrption as  aws_access_key_id . <br>
- Similarly create credentials for aws_secret_access_key . These to key will be apssed as Environmental variable in jenkins pipeline to connect to EKS cluster.
 









    
    
    


   

   
   
   

