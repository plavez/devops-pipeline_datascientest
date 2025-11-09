pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "plavez/devops-pipeline"   // укажи свой репозиторий DockerHub
        DOCKER_CREDS = "58682082-4fbd-4e02-a3ab-4a11c0c78ac0"  // ID твоих DockerHub credentials
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker image') {
            steps {
                script {
                    sh '''
                    echo "Building Docker image..."
                    docker build -t $DOCKER_IMAGE:latest .
                    '''
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh '''
                        echo "Logging into DockerHub..."
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        echo "Pushing image to DockerHub..."
                        docker push $DOCKER_IMAGE:latest
                        docker logout
                        '''
                    }
                }
            }
        }
    }
}
