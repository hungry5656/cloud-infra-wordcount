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

        // stage('Install Java') {
        //     steps {
        //         sh '''
                    
        //             sudo apt-get update
        //             sudo apt-get install -y openjdk-8-jdk
        //             export JAVA_HOME=$(readlink -f /usr/bin/java | sed "s:/bin/java::")
        //             export PATH=$JAVA_HOME/bin:$PATH

        //             java -version
        //             javac -version
        //         '''
        //     }
        // }

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
                    if ! command -v hadoop &> /dev/null
                    then
                        sudo apt-get install -y wget tar
                        wget --version
                        tar --version
                        sudo rm -rf /usr/local/hadoop
                        if [ ! -f hadoop-3.4.1.tar.gz ]; then
                            wget --no-verbose https://dlcdn.apache.org/hadoop/common/hadoop-3.4.1/hadoop-3.4.1.tar.gz
                        fi
                        tar -zxf hadoop-3.4.1.tar.gz > /dev/null 2>&1
                        sudo mv hadoop-3.4.1 /usr/local/hadoop
                        export HADOOP_HOME=/usr/local/hadoop
                        export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
                    fi

                    hadoop version
                    cd cloud-infra-wordcount
                    mkdir -p out
                    hadoop classpath
                    javac -classpath `hadoop classpath` -d . WordCount.java
                    jar cvf wc.jar -C out/ .
                    echo "Compiled wc.jar successfully."
                    cd ..
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
