pipeline {
    agent any

    environment {
        DOCKER_HUB_CRED = 'dockerhub-credentials'
        DOCKER_IMAGE    = 'sumedh1299/calculator-app'
        IMAGE_TAG       = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                echo 'Validating application files...'
                sh '''
                    test -f index.html || exit 1
                    which xmllint > /dev/null 2>&1 && xmllint --html --noout index.html || echo "Basic file check passed."
                '''
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
                sh "docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest"
            }
        }

        stage('Push Image') {
            steps {
                echo 'Publishing image to Docker Hub...'
                withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CRED}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Minikube Deployment') {
            steps {
                echo 'Applying Kubernetes manifests...'
                sh '''
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    kubectl set image deployment/calculator-deployment calculator-container=${DOCKER_IMAGE}:${IMAGE_TAG}
                '''
            }
        }

        stage('Verification') {
            steps {
                echo 'Verifying deployment status...'
                sh '''
                    kubectl rollout status deployment/calculator-deployment
                    kubectl get pods -l app=calculator
                    kubectl get svc calculator-service
                '''
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
        success {
            echo "Pipeline executed successfully and application deployed!"
        }
        failure {
            echo "Pipeline failed. Check stage logs for errors."
        }
    }
}
