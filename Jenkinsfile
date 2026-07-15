pipeline {
    agent any

    environment {
        APP_DIR = "${WORKSPACE}"
        BACKEND_IMAGE = "tejaguddeti/lms-be:${BUILD_NUMBER}"
        FRONTEND_IMAGE = "tejaguddeti/lms-fe:${BUILD_NUMBER}"
    }

    stages {

        stage('Build Backend Image') {
            steps {
                dir("${APP_DIR}/api") {
                    sh '''
                    cat > .env <<EOF
MODE=dev
PORT=8080
DATABASE_URL=postgresql://postgres:teja123@postgres-cluster-ip-service:5432/postgres
EOF

                    docker build -t ${BACKEND_IMAGE} .
                    '''
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                dir("${APP_DIR}/webapp") {
                    sh '''
                    cat > .env <<EOF
VITE_API_URL=http://lms-be-service:8080/api
EOF

                    docker build -t ${FRONTEND_IMAGE} .
                    '''
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Backend Image') {
            steps {
                sh '''
                docker push ${BACKEND_IMAGE}
                '''
            }
        }

        stage('Push Frontend Image') {
            steps {
                sh '''
                docker push ${FRONTEND_IMAGE}
                '''
            }
        }

        stage('Update Kubernetes YAML') {
            steps {
                sh '''
                sed -i "s|IMAGE_BACKEND|${BACKEND_IMAGE}|g" k8s/be-deploy.yaml
                sed -i "s|IMAGE_FRONTEND|${FRONTEND_IMAGE}|g" k8s/fe-deploy.yaml
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f k8s/
                '''
            }
        }

        stage('Wait for Rollout') {
            steps {
                sh '''
                kubectl rollout status deployment/lms-be
                kubectl rollout status deployment/lms-fe
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                kubectl get pods
                kubectl get svc
                kubectl get deployments
                '''
            }
        }
    }

    post {
        success {
            echo 'Application Successfully Deployed to Kubernetes.'
        }

        failure {
            echo 'Pipeline Failed.'
        }

        always {
            sh '''
            docker image prune -f
            '''
        }
    }
}
