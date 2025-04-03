pipeline {
    agent any
    environment {
        REPO_URL = 'https://github.com/hungry5656/cloud-infra-wordcount.git'
    }
    stages { 
        stage('Hello') {
            steps {
                echo 'Hello World'
                // sh 'sudo rm -rf wc.jar out'
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
                    else
                        echo "Java 8 is already installed."
                    fi



                    # if ! command -v hadoop &> /dev/null
                    # then
                    #     sudo apt-get install -y wget tar
                    #     wget --version
                    #     tar --version
                    #     if [ -d /usr/local/hadoop ]; then
                    #         export HADOOP_HOME=/usr/local/hadoop
                    #         export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
                    #     else
                    #         sudo rm -rf /usr/local/hadoop
                    #         if [ ! -f hadoop-3.4.1.tar.gz ]; then
                    #             wget --no-verbose https://dlcdn.apache.org/hadoop/common/hadoop-3.4.1/hadoop-3.4.1.tar.gz
                    #         fi
                    #         tar -zxf hadoop-3.4.1.tar.gz > /dev/null 2>&1
                    #         sudo mv hadoop-3.4.1 /usr/local/hadoop
                    #         export HADOOP_HOME=/usr/local/hadoop
                    #         export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
                    #     fi
                    # fi

                    # hadoop version
                    # hadoop classpath
                    # javac -classpath `hadoop classpath` -d . WordCount.java
                    # jar cf wc.jar WordCount*.class
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
                        gsutil rm -rf gs://${BUCKET_NAME}/output
                        gsutil cp input.txt gs://$BUCKET_NAME

                        # Submit the job to Dataproc
                        gcloud dataproc jobs submit hadoop \
                            --cluster=hadoop-cluster \
                            --region=us-west1 \
                            --jar=gs://${BUCKET_NAME}/wc.jar \
                            -- WordCount gs://${BUCKET_NAME}/input.txt gs://${BUCKET_NAME}/output
                    '''
                }
            }
        }
    }
}
