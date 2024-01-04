pipeline {
    agent any
    
    environment {
        registry = "852524605641.dkr.ecr.us-east-2.amazonaws.com/jenkins_project"
    }
    
    parameters {
        string(name: 'VERSION', defaultValue: '1.21', description: 'Version tag parameter')
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
                        docker build -t 852524605641.dkr.ecr.us-east-2.amazonaws.com/jenkins_project:"${VERSION}" .
                    '''
                }
            }
        }
        
        stage("Push Image to ECR") {
            steps {
                script {
                    sh '''
                        aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 852524605641.dkr.ecr.us-east-2.amazonaws.com
                        docker push 852524605641.dkr.ecr.us-east-2.amazonaws.com/jenkins_project:"${VERSION}"
                    '''
                }
            }
        }
        
        stage('Invoke Sub Pipeline') {
            steps {
                build job: 'jenkins-sub-pipeline', parameters: [
                    string(name: 'VERSION', value: "${params.VERSION}")
                ]
            }
        }

        stage('Calculate Next Version') {
            steps {
                script {
                    // 현재 버전을 가져오기
                    def currentVersion = params.VERSION

                    // 다음 버전 계산 (예: 1.14 -> 1.15)
                    def nextVersion = calculateNextVersion(currentVersion)

                    // 새로운 빌드를 위한 파라미터 설정
                    properties([
                        parameters([
                            string(name: 'VERSION', value: nextVersion)
                        ])
                    ])

                    // 업데이트된 버전 출력
                    echo "Next version: ${nextVersion}"
                }
            }
        }
    }
}

def calculateNextVersion(currentVersion) {
    // 여기에서 원하는 로직에 따라 다음 버전을 계산
    // 예: 1.14 -> 1.15
    def parts = currentVersion.tokenize('.')
    def major = parts[0] as Integer
    def minor = parts[1] as Integer
    def nextMinor = minor + 1
    return "${major}.${nextMinor}"
}
