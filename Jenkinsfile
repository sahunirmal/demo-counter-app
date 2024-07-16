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
    }             
}  
            

