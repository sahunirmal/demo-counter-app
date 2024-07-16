pipeline{
    agent any
    tools{
        maven 'mymaven'
    }
    stages {
        stage('Git Checkout'){
            steps{
                    git branch: 'master', url: 'https://github.com/sahunirmal/demo-counter-app.git'
            }
        }
        stage('UNIT testing'){
            steps{
                    sh 'mvn test'
            }
        }
        stage('Integration testing'){
            steps{       
                    sh 'mvn verify -DskipUnitTests' 
            }
        }
        stage('Maven build'){
            steps{                    
                    sh 'mvn clean install'                
            }
        }
        stage('SonarQube analysis'){          
            steps{               
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-api') { 
                        sh 'mvn clean package sonar:sonar'
                    }
                   } 
                }
            }   
        stage('Quality Gtae status'){          
            steps{               
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-api'
                }
            } 
                
        }
        stage('Upload jar file to nexus'){
            steps{
                script{
                    def readPomVersion = readMavenPom file: 'pom.xml'
                    def nexusRepo = readMavenPom.version.endsWith("SNAPSHOT") ? "demoapp-snapshot" : "demoapp-release"
                    nexusArtifactUploader artifacts:
                     [
                        [
                            artifactId: 'springboot',
                            classifier: '', file: 'target/Uber.jar',
                            type: 'jar'
                        ]
                     ], 
                     credentialsId: 'nexus-auth', 
                     groupId: 'com.example', 
                     nexusUrl: '44.192.128.218:8081', 
                     nexusVersion: 'nexus3', 
                     protocol: 'http', 
                     repository: nexusRepo,
                     version: "${readPomVersion.version}"
                }
            }
        }
    }             
}  
            

