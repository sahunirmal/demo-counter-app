pipeline {
    agent any
    tools {
        maven 'mymaven'
    }
    parameters {
        choice(name: 'action', choices: 'create\ndestroy\ndestroyekscluster', description: 'Create/Update or destroy the EKS cluster')
        string(name: 'cluster', defaultValue: 'demo-cluster', description: 'EKS cluster name')
        string(name: 'region', defaultValue: 'us-east-1', description: 'EKS cluster region')
    }
    environment {
        ACCESS_KEY = credentials('aws_access_key_id')
        SECRET_KEY = credentials('aws_secret_access_key')
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/sahunirmal/demo-counter-app.git'
            }
        }
        stage('Unit Testing') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Integration Testing') {
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
        stage('Maven Build') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-api') {
                        sh 'mvn clean package sonar:sonar'
                    }
                }
            }
        }
        stage('Quality Gate Status') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-api'
                }
            }
        }
        stage('Upload Jar File to Nexus') {
            steps {
                script {
                    def readPomVersion = readMavenPom file: 'pom.xml'
                    def nexusRepo = readPomVersion.version.endsWith("SNAPSHOT") ? "demoapp-snapshot" : "demoapp-release"
                    nexusArtifactUploader artifacts: [
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
        stage('Docker Image Build') {
            steps {
                script {
                    sh 'docker image build -t $JOB_NAME:v1.$BUILD_ID .'
                    sh 'docker image tag $JOB_NAME:v1.$BUILD_ID nirmalendusahu/$JOB_NAME:v1.$BUILD_ID'
                    sh 'docker image tag $JOB_NAME:v1.$BUILD_ID nirmalendusahu/$JOB_NAME:latest'
                }
            }
        }
        stage('Push Image to DockerHub') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker_creds', variable: 'docker_hub_cred')]) {
                        sh 'docker login -u nirmalendusahu -p ${docker_hub_cred}'
                        sh 'docker image push nirmalendusahu/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker image push nirmalendusahu/$JOB_NAME:latest'
                    }
                }
            }
        }
        stage('EKS Connect') {
            steps {
                sh """
                   aws configure set aws_access_key_id "$ACCESS_KEY"
                   aws configure set aws_secret_access_key "$SECRET_KEY"
                   aws configure set region "${params.region}"
                   aws eks --region ${params.region} update-kubeconfig --name ${params.cluster}
                """
            }
        }
        stage('EKS Deployment') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    def apply = false
                    try {
                        input message: 'Please confirm to initiate the deployments', ok: 'Ready to apply the config'
                        apply = true
                    } catch (err) {
                        apply = false
                        currentBuild.result = 'UNSTABLE'
                    }
                    if (apply) {
                        sh 'kubectl apply -f .'
                    }
                }
            }
        }
        stage('Delete Deployments') {
            when { expression { params.action == 'destroy' } }
            steps {
                script {
                    def destroy = false
                    try {
                        input message: 'Please confirm to delete the deployments', ok: 'Ready to destroy the config'
                        destroy = true
                    } catch (err) {
                        destroy = false
                        currentBuild.result = 'UNSTABLE'
                    }
                    if (destroy) {
                        sh 'kubectl delete -f .'
                    }
                }
            }
        }
    }            
}
