pipeline {
    agent any
    environment {
        REPO_URL = 'https://github.com/hungry5656/cloud-infra-wordcount.git'
    }
    stages { 
        stage('Hello') {
            steps {
                echo 'Hello World'
                sh 'sudo rm -rf hadoop-3.4.1/'
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
                withSonarQubeEnv('my-sonar') { 
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) { 
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=course-project-option-i-hungry5656 \
                            -Dsonar.login=$SONAR_TOKEN \
                            -Dsonar.coverage.exclusions=**/*.java \
                            -Dsonar.coverage.minimumCoverage=10
                        """
                    }
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

        stage('Compile Project') {
            steps {
                sh '''
                    export HADOOP_HOME=/usr/local/hadoop
                    export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH

                    sudo apt install -y maven
                    
                    if [ ! -d /usr/local/java/zulu8.84.0.15-ca-jdk8.0.442-linux_x64 ]
                    then
                        echo "Java 8 is not installed. Installing Java 8..."
                        wget --no-verbose https://cdn.azul.com/zulu/bin/zulu8.84.0.15-ca-jdk8.0.442-linux_x64.tar.gz
                        tar -zxf zulu8.84.0.15-ca-jdk8.0.442-linux_x64.tar.gz > /dev/null 2>&1
                        sudo mv zulu8.84.0.15-ca-jdk8.0.442-linux_x64 /usr/local/java
                        rm -rf zulu8.84.0.15-ca-jdk8.0.442-linux_x64.tar.gz
                    else
                        echo "Java 8 is already installed."
                    fi
                    mvn clean package > /dev/null 2>&1
                    mv target/wordcount-1.0-SNAPSHOT-jar-with-dependencies.jar wc.jar
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

                        BUCKET_NAME=$(gcloud storage buckets list --filter="name:dataproc-staging-us-west1-*" --format="value(name)")



                        echo "Bucket Name: $BUCKET_NAME"

                        gsutil cp wc.jar gs://$BUCKET_NAME
                        gsutil cp input.txt gs://$BUCKET_NAME

                        # Submit the job to Dataproc
                        TIMESTAMP=$(date +%Y%m%d%H%M%S)
                        OUTPUT_FOLDER="output-${TIMESTAMP}"
                        gcloud dataproc jobs submit hadoop \
                            --cluster=hadoop-cluster \
                            --region=us-west1 \
                            --jar=gs://${BUCKET_NAME}/wc.jar \
                            -- WordCount gs://${BUCKET_NAME}/input.txt gs://${BUCKET_NAME}/${OUTPUT_FOLDER}
                        
                        gsutil -m cp -r gs://${BUCKET_NAME}/${OUTPUT_FOLDER} .
                        echo "Results copied to local folder: ${OUTPUT_FOLDER}"

                        echo "\n\nResult of Hadoop Job:"
                        for file in $(find ${OUTPUT_FOLDER} -type f); do
                            cat $file
                        done

                        rm -rf ${OUTPUT_FOLDER}
                    '''
                }
            }
        }
    }
}
