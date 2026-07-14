pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                deleteDir()

                git branch: 'main',
                    url: 'https://github.com/devendra540/my-devops-project.git'
            }
        }

        stage('Build Application') {
            steps {
                sh '''
                    mvn clean package -DskipTests
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
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
                    -t my-devops-project:latest \
                    .
                '''
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
                    echo "Waiting for Spring Boot to start..."
                    sleep 15

                    docker ps --filter "name=springboot-app"

                    curl --fail \
                         --retry 5 \
                         --retry-delay 5 \
                         http://localhost:8082
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline with SonarQube completed successfully.'
            echo 'Spring Boot application is running on port 8082.'
        }

        failure {
            echo 'Pipeline failed. Showing container information.'

            sh '''
                docker ps -a || true
                docker logs springboot-app --tail 100 2>/dev/null || true
            '''
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
