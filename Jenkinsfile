
pipeline {
    agent any
    environment{
        APP_DIR = "${WORKSPACE}"
        BACKEND_IMAGE="tejaguddeti/lms-be"
        FRONTEND_IMAGE="tejaguddeti/lms-fe"
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
        stage('Deploy Backend') {
            steps {
                sh ''' docker network create lms-network || true
                       docker rm -f lms-db || true
                       docker rm -f lms-be || true
                      
                docker run -dt --name lms-db --network lms-network -e POSTGRES_PASSWORD=Login@123 postgres
                 sleep 20
                 docker run -dt --name lms-be --network lms-network -p 8081:8080 ${BACKEND_IMAGE} '''
                
            }
        }
        stage('Build Frontend Image') {
            steps {
                sh ''' cd ${APP_DIR}/webapp
                 cat > .env <<EOF 
VITE_API_URL=http://3.26.199.100:8081/api
EOF
                    docker build -t ${FRONTEND_IMAGE} . '''
            }
        }
        stage('Deploy Frontend') {
            steps {
                sh ''' docker rm -f lms-fe || true
                docker run -dt  --name lms-fe --network lms-network -p 80:80 ${FRONTEND_IMAGE} '''
            }
        }
        stage('verify') {
            steps {
                sh ''' docker ps -a
                docker logs lms-db || true
                docker logs lms-be || true
                docker logs lms-fe || true
                curl http://localhost:8080/api || true '''
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
