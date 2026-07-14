pipeline {
    agent any

    tools {
        maven 'Maven'
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
                sh 'docker build -t my-devops-project:latest .'
            }
        }

        stage('Remove Old Container') {
            steps {
                sh 'docker rm -f springboot-app 2>/dev/null || true'
            }
        }

        stage('Run New Container') {
            steps {
                sh '''
                    docker run -d \
                    --name springboot-app \
                    -p 8082:8080 \
                    my-devops-project:latest
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'docker ps'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
