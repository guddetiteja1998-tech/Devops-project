
pipeline {
    agent any
    environment {
    APP_DIR = "${WORKSPACE}"
      DOCKER_USERNAME = "tejaguddeti"
      BACKEND_IMAGE = "tejaguddeti/lms-be:${BUILD_NUMBER}"
      FRONTEND_IMAGE = "tejaguddeti/lms-fe:${BUILD_NUMBER}"
}
    }

    stages {
        stage('Build Backend Image') {
            steps {
                sh ''' cd ${APP_DIR}/api
                cat > .env <<EOF
MODE=dev
PORT=8080
DATABASE_URL=postgresql://postgres:Login@123@lms-db:5432/postgres
EOF     
                       docker build -t ${BACKEND_IMAGE} . '''
            }
        }
        stage('Docker Login') {
            steps {
            withCredentials([usernamePassword(
              credentialsId: 'dockerhub',
              usernameVariable: 'DOCKER_USER',
              passwordVariable: 'DOCKER_PASS'
        )]) {
                sh ''' echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin '''
                
            }
        }
        stage('Push Backend Image') {
            steps {
                sh ''' docker push ${BACKEND_IMAGE} '''
            }
        }
        stage('Push Frontend Image') {
            steps {
                sh ''' docker push ${FRONTEND_IMAGE} '''
            }
        }
        stage('verify') {
            steps {
                sh ''' docker ps -a
                docker logs lms-db || true
                docker logs lms-be || true
                docker logs lms-fe || true
                curl http://localhost:8081/api || true '''
            }
        }
        
    }

    post {
        success {
            echo "Application deployed successfully."
        }

        failure {
            echo "Deployment Failed."
        }
    }
}
