pipeline {
    agent any
    
    environment {
        registry = "852524605641.dkr.ecr.us-east-2.amazonaws.com/jenkins_project"
    }

    
    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/shimmins/docker-spring-boot']])
            }
        }
        
        stage("Build Jar") {
            steps {
                sh "mvn clean install"
            }
        }
        
        stage("Build Image") {
            steps {
                script {
                    sh '''
                        docker build -t 852524605641.dkr.ecr.us-east-2.amazonaws.com/jenkins_project:$BUILD_NUMBER .
                    '''
                }
            }
        }
        
        stage("Push Image to ECR") {
            steps {
                script {
                    sh '''
                        aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 852524605641.dkr.ecr.us-east-2.amazonaws.com
                        docker push 852524605641.dkr.ecr.us-east-2.amazonaws.com/jenkins_project:$BUILD_NUMBER
                    '''
                }
            }
        }
        
        stage('Invoke Sub Pipeline') {
            steps {
                build job: 'jenkins-sub-pipeline', parameters: [
                    string(name: 'VERSION', value: "${BUILD_NUMBER}")
                ]
            }
        }

