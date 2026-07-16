pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        AWS_REGION      = 'us-east-1'
        AWS_ACCOUNT_ID  = '115154236409'
        ECR_REPOSITORY  = 'my-devops-project'
        ECR_REGISTRY    = '115154236409.dkr.ecr.us-east-1.amazonaws.com'
        EKS_CLUSTER     = 'devops-eks-cluster'
        KUBECONFIG      = '/var/lib/jenkins/.kube/config'
        IMAGE_NAME      = "${ECR_REGISTRY}/${ECR_REPOSITORY}"
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
             stage('Deploy Application to EKS') {
             steps {
              sh '''
             kubectl delete deployment springboot-app --ignore-not-found=true

                kubectl create deployment springboot-app \
                    --image=${IMAGE_NAME}:${BUILD_NUMBER}
            '''
    }
}
        stage('Deploy Application to EKS') {
            steps {
                sh '''
                    kubectl create deployment springboot-app \
                    --image=${IMAGE_NAME}:${BUILD_NUMBER} \
                    --dry-run=client \
                    -o yaml | kubectl apply -f -

                    kubectl set image \
                    deployment/springboot-app \
                    springboot-app=${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Create LoadBalancer Service') {
            steps {
                sh '''
                    kubectl expose deployment springboot-app \
                    --type=LoadBalancer \
                    --port=80 \
                    --target-port=8080 \
                    --name=springboot-service \
                    --dry-run=client \
                    -o yaml | kubectl apply -f -
                '''
            }
        }

        stage('Verify EKS Deployment') {
            steps {
                sh '''
                    kubectl rollout status \
                    deployment/springboot-app \
                    --timeout=300s

                    kubectl get deployments
                    kubectl get pods
                    kubectl get service springboot-service
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
            echo 'Docker image pushed to ECR and application deployed to EKS.'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins Console Output.'
        }

        always {
            sh '''
                docker image prune -f || true
            '''
        }
    }
}
