pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        AWS_REGION   = 'us-east-1'
        ECR_REGISTRY = '115154236409.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPOSITORY = 'my-devops-project'
        EKS_CLUSTER  = 'devops-eks-cluster'
        KUBECONFIG   = "${WORKSPACE}/.kubeconfig"
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
                    -t ${ECR_REPOSITORY}:latest .
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

        stage('Tag and Push Image to ECR') {
            steps {
                sh '''
                    docker tag \
                    ${ECR_REPOSITORY}:${BUILD_NUMBER} \
                    ${ECR_REGISTRY}/${ECR_REPOSITORY}:${BUILD_NUMBER}

                    docker tag \
                    ${ECR_REPOSITORY}:latest \
                    ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest

                    docker push \
                    ${ECR_REGISTRY}/${ECR_REPOSITORY}:${BUILD_NUMBER}

                    docker push \
                    ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest
                '''
            }
        }

        stage('Configure EKS Access') {
            steps {
                sh '''
                    aws eks update-kubeconfig \
                    --name ${EKS_CLUSTER} \
                    --region ${AWS_REGION} \
                    --kubeconfig ${KUBECONFIG}

                    kubectl get nodes
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    kubectl set image \
                    deployment/springboot-app \
                    springboot-app=${ECR_REGISTRY}/${ECR_REPOSITORY}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Verify Kubernetes Deployment') {
            steps {
                sh '''
                    kubectl rollout status \
                    deployment/springboot-app \
                    --timeout=300s

                    kubectl get pods
                    kubectl get service springboot-service
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully and application deployed to EKS.'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins Console Output.'
        }
    }
}
