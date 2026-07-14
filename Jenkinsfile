pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/devendra540/my-devops-project.git'
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t my-devops-project:latest .'
            }
        }

        stage('Remove Old Container') {
            steps {
                sh '''
                    docker rm -f springboot-app 2>/dev/null || true
                '''
            }
        }

        stage('Run New Container') {
            steps {
                sh '''
                    docker run -d \
                    -p 8082:8080 \
                    --name springboot-app \
                    --restart unless-stopped \
                    my-devops-project:latest
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    sleep 10
                    docker ps
                    curl -f http://localhost:8082
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline Completed Successfully'
            echo 'Spring Boot application deployed on port 8082'
        }

        failure {
            echo 'Pipeline Failed'
            sh 'docker ps -a || true'
            sh 'docker logs springboot-app --tail 100 || true'
        }

        always {
            echo 'Pipeline execution completed'
        }
    }
}
