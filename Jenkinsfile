pipeline {
    agent any
    stages { 
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
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
              withSonarQubeEnv(credentialsId: 'sonarqube-token', installationName: 'my-sonar') {
                sh "${scannerHome}/bin/sonar-scanner"
              }
            }
        }
    }
}
