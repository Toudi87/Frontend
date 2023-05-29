pipeline {
    agent {
      label 'agent'
    }
    environment {
        scannerHome = tool 'SonarQube'
    }
    stages {
        stage('clone repo') {
            steps {
                checkout scm
            }
        }
      stage('Unit tests') {
        steps {
           sh "pip3 install -r requirements.txt"
           sh "python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml"
        }
      }
      stage('Start SonarQube') {
        steps {
            withSonarQubeEnv('SonarQube') {
                sh "${scannerHome}/bin/sonar-scanner"
            }
                
            timeout(1) {
                waitForQualityGate abortPipeline: true
            }
        }
      }
    }
    
    post {
        always {
            junit test-results/*.xml
            cleanWs()
        }
    }
    
}