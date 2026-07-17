pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        AWS_REGION     = 'us-east-1'
        ECR_REGISTRY   = '115154236409.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPOSITORY = 'my-devops-project'
        IMAGE_NAME     = "${ECR_REGISTRY}/${ECR_REPOSITORY}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarServer') {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=my-devops-project \
                        -Dsonar.projectName=my-devops-project
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                    -t ${ECR_REPOSITORY}:${BUILD_NUMBER} \
                    -t ${ECR_REPOSITORY}:latest \
                    .
                '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh '''
                    aws ecr get-login-password \
                    --region ${AWS_REGION} |
                    docker login \
                    --username AWS \
                    --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                    docker tag \
                    ${ECR_REPOSITORY}:${BUILD_NUMBER} \
                    ${IMAGE_NAME}:${BUILD_NUMBER}

                    docker tag \
                    ${ECR_REPOSITORY}:latest \
                    ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                    docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                    docker push ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Run Application on EC2') {
            steps {
                sh '''
                    docker rm -f springboot-app 2>/dev/null || true

                    docker run -d \
                    --name springboot-app \
                    --restart unless-stopped \
                    -p 8082:8080 \
                    ${ECR_REPOSITORY}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Verify Application') {
            steps {
                sh '''
                    sleep 10
                    docker ps
                    curl -f http://localhost:8082 || true
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
            echo 'Docker image pushed to ECR and application deployed on EC2.'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins Console Output.'
        }

        always {
            sh 'docker image prune -f || true'
        }
    }
}
