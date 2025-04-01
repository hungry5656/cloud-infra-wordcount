pipeline {
    agent any 
    
    stages { 
        stage('SCM Checkout') {
            steps{
                git branch: 'main',
                url: 'https://github.com/hungry5656/cloud-infra-wordcount'
            }
        }
        stage('Run SonarQube') {
            environment {
                scannerHome = tool 'sonarQube scanner 1';
            }
            steps {
              withSonarQubeEnv(credentialsId: 'sonar-secret', installationName: 'sonarQube install') {
                sh "${scannerHome}/bin/sonar-scanner"
              }
            }
        }
    }
}