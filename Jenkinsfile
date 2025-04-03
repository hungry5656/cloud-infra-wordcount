pipeline {
    agent any
    environment {
        REPO_URL = 'https://github.com/hungry5656/cloud-infra-wordcount.git'
        JAVA_HOME = '/usr/lib/jvm/java-11-openjdk-amd64'
    }
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
        stage('Clone Repository') {
            steps {
                sh '''
                    rm -rf cloud-infra-wordcount
                    git clone ${REPO_URL}
                '''
            }
        }

        stage('Compile Project') {
            steps {
                sh '''
                    cd cloud-infra-wordcount
                    mkdir -p out
                    javac -classpath `hadoop classpath` -d . WordCount.java
                    jar cvf wc.jar -C out/ .
                    echo "Compiled wc.jar successfully."
                '''
            }
        }
        stage('Submit Hadoop Job') {
            steps {
                withCredentials([file(credentialsId: 'a123', variable: 'G_CREDENTIALS')]) {
                    sh '''
                        gcloud --version

                        gcloud auth activate-service-account --key-file="${G_CREDENTIALS}"

                        gcloud config set project cloudinfra-project-5656

                        BUCKET_NAME=$(gcloud storage buckets list --filter="name:dataproc-staging-us-central1-*" --format="value(name)")

                        echo "Bucket Name: $BUCKET_NAME"

                        gsutil cp cloud-infra-wordcount/out/wc.jar $BUCKET_NAME
                        gsutil cp cloud-infra-wordcount/input.txt $BUCKET_NAME

                        # Submit the job to Dataproc
                        gcloud dataproc jobs submit hadoop \
                            --cluster=hadoop-cluster \
                            --region=us-west1 \
                            --jar=${BUCKET_NAME}/wc.jar \
                            -- WordCount ${BUCKET_NAME}/input.txt ${BUCKET_NAME}/output
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
