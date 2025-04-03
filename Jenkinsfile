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
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Submit Hadoop Job') {
            steps {
                withCredentials([file(credentialsId: 'a123', variable: 'G_CREDENTIALS')]) {
                    sh '''
                        gcloud --version
                        gcloud auth activate-service-account --key-file="${G_CREDENTIALS}"
                    '''
                }
                
                
                //sh "gsutil -m rm -r gs://cloudinfra-project-5656/output || true"

                // sh '''
                //    gcloud dataproc jobs submit hadoop \
                //       --cluster=hadoop-cluster \
                //       --region=us-west1 \
                //       --jar=gs://dataproc-examples-us/wordcount/wordcount.jar \
                //       -- wordcount gs://cloudinfra-project-5656/input gs://cloudinfra-project-5656/output
                // '''
            }
        }
    }
}
