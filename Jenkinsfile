def imageName="192.168.44.44:8082/docker-local/frontend"
def dockerRegistry="https://192.168.44.44:8082"
def registryCredentials="artifactory"
def dockerTag=""

pipeline {
    agent {
        label 'agent'
    }
    environment {
        scannerHome = tool 'SonarQube'
    }
    stages {
        stage('Git checkout Fronted') {
            steps {
                checkout scm 
            }
        }    
        stage('Run Unit test') {
             steps {
                echo "Run Unit tests"
                sh """
                pip3 install -r requirements.txt
                python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml
                """
            }
        }
        stage('Sonarqube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
                 timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build application image') {
            steps {
                script {
                    dockerTag="RC-${env.BUILD_ID}" // .${env.GIT_COMMIT.take(7)}"
                    applicationImage=docker.build("$imageName:$dockerTag")
                }
            }
        }
        stage('Push application image to artifactory') {
            steps {
                script {
                    docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                        applicationImage.push("latest")
                    }
                }
            }
        }
    }
    post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
    }
}