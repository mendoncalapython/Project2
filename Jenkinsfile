pipeline {

    agent any

    environment {
        DOCKER_IMAGE = "mendoncalanpython/analytics-app"
        DOCKER_TAG = "latest"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                    -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'mendoncalanpython',
                        passwordVariable: 'dckr_pat_xtObSQt_PKuaJDV-hWWILnIkySs'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                        -u "$DOCKER_USERNAME" \
                        --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl rollout status deployment/analytics-app
                    kubectl get pods
                    kubectl get svc analytics-service
                '''
            }
        }
    }

    post {

        success {
            echo 'Application successfully deployed!'
        }

        failure {
            echo 'Pipeline failed!'
        }

        always {
            sh 'docker logout || true'
        }
    }
}
