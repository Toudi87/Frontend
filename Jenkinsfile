// def imageName="192.168.44.44:8082/docker_registry/frontend"
// def dockerRegistry="https://192.168.44.44:8082"
// def registryCredentials="artifactory"
// def dockerTag=""

def imageName="darek87/frontend"
def dockerRegistry=""
def registryCredentials="docker_hub"
def dockerTag=""


pipeline {
    agent {
        label 'agent'
    }
    
    environment {
        scannerHome = tool 'SonarQube'
    }
    
    stages {
        stage('Get Code') {
            steps {
                checkout scm
            }
        }
        stage('Unit tests') {
            steps {
                sh 'pip3 install -r requirements.txt'
                sh 'python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml'
            }
        }
        stage('Start SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
                timeout(1) {
                    waitForQualityGate false
                }
            }
        }
        stage('Build application image') {
            steps {
                script{
                    //dockerTag = "RC-${env.BUILD_ID}.${env.GIT_COMMIT.take(7)}"
                    dockerTag = "RC-${env.BUILD_ID}"
                    applicationImage = docker.build("$imageName:$dockerTag", ".")
                }
            }
        }
        stage('Pushing application to Artifactory/DockerHub') {
            steps {
                script{
                    docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                        applicationImage.push()
                        applicationImage.push('latest')
                    }
                }
            }
        }
        stage ('Push to repo') {
            steps {
                dir('ArgoCD') {
                    withCredentials([gitUsernamePassword(credentialsId: 'git', gitToolName: 'Default')]) {
                        git branch: 'master', url: 'https://github.com/Toudi87/argo.git'
                        sh """ cd backend
                        git config --global user.email "dariusz.scibior@gmail.com"
                        git config --global user.name "Darek"
                        sed -i "s#$imageName.*#$imageName:$dockerTag#g" deployment.yaml
                        git commit -am "Set new $dockerTag tag."
                        git diff
                        git push origin master
                        """
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