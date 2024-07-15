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
