pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "plavez/devops-pipeline"
        DOCKER_CREDS = "58682082-4fbd-4e02-a3ab-4a11c0c78ac0"   // ID DockerHub credentials
        KUBECONFIG_CRED = "kubeconfig-main"                     // kubeconfig credential
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker image') {
            steps {
                sh '''
                  echo "Building image..."
                  docker build -t ${DOCKER_IMAGE}:latest .
                  docker tag ${DOCKER_IMAGE}:latest ${DOCKER_IMAGE}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                      docker push ${DOCKER_IMAGE}:latest
                      docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                      docker logout
                    '''
                }
            }
        }

        stage('Deploy to dev (auto)') {
            steps {
                withCredentials([file(credentialsId: env.KUBECONFIG_CRED, variable: 'KCFG')]) {
                    sh '''
                      export KUBECONFIG="$KCFG"
                      kubectl apply -f k8s/base/dev/ns.yaml
                      kubectl -n dev apply -f k8s/base/deployment.yaml
                      kubectl -n dev apply -f k8s/base/service.yaml
                      kubectl -n dev set image deployment/devops-web web=${DOCKER_IMAGE}:${BUILD_NUMBER}
                      kubectl -n dev rollout status deployment/devops-web
                    '''
                }
            }
        }

        stage('Deploy to prod (manual, only main)') {
            when { branch 'main' }
            steps {
                input message: 'Deploy to PRODUCTION?'
                withCredentials([file(credentialsId: env.KUBECONFIG_CRED, variable: 'KCFG')]) {
                    sh '''
                      export KUBECONFIG="$KCFG"
                      kubectl apply -f k8s/base/prod/ns.yaml
                      kubectl -n prod apply -f k8s/base/deployment.yaml
                      kubectl -n prod apply -f k8s/base/service.yaml
                      kubectl -n prod set image deployment/devops-web web=${DOCKER_IMAGE}:${BUILD_NUMBER}
                      kubectl -n prod rollout status deployment/devops-web
                    '''
                }
            }
        }
    }
}
