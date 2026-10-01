pipeline {

    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = 'java-demo-ecr'
        ECR_URI = '581245317342.dkr.ecr.ap-south-1.amazonaws.com/java-demo-ecr'
        EKS_CLUSTER = 'java-demo-eks'

        JAVA_HOME = '/usr/lib/jvm/java-21-amazon-corretto.x86_64'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Java & Maven') {
            steps {
                sh '''
                    echo "Java Version:"
                    java -version

                    echo "Maven Version:"
                    mvn -version
                '''
            }
        }

        stage('Maven Build') {
            steps {
                sh '''
                    mvn clean package -DskipTests
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t java-demo-cicd:${BUILD_NUMBER} .
                '''
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${ECR_URI}
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                    docker tag java-demo-cicd:${BUILD_NUMBER} ${ECR_URI}:${BUILD_NUMBER}

                    docker push ${ECR_URI}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    aws eks update-kubeconfig \
                      --region ${AWS_REGION} \
                      --name ${EKS_CLUSTER}

                    kubectl set image deployment/java-demo-cicd \
                      java-demo=${ECR_URI}:${BUILD_NUMBER}

                    kubectl rollout status deployment/java-demo-cicd
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed. Check the console output.'
        }
    }
}
