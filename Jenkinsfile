
pipeline {
    agent any
    environment{
        APP_DIR="/home/ubuntu/DEVOPS-PROJECT-MAIN"
        BACKEND_IMAGE="tejaguddeti/lms-be"
        FRONTEND_IMAGE="tejaguddeti/lms-fe"
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                sh '''rm -rf ${APP_DIR}'''
            }
        }
        stage('cloning a repo') {
            steps {
                sh ''' cd /home/ubuntu 
                https://github.com/guddetiteja1998-tech/Devops-project.git '''
            }
        }
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
                sh ''' docker rm -f lms-be || true
                docker container run -dt --name lms-be --network bridge -p 8080:8080 ${BACKEND_IMAGE} '''
                
            }
        }
        stage('Build Frontend Image') {
            steps {
                sh ''' cd {APP_DIR}/webapp
                 cat > .env <<EOF 
VITE_API_URL=http://YOUR_PUBLIC_IP:8080/api
EOF
                    docker build -t ${FRONTEND-IMAGE}. '''
            }
        }
        stage('Deploy Frontend') {
            steps {
                sh ''' docker rm -f lms-be || true
                docker container run -dt --name lms-fe --network bridge -p 80:80 ${FRONTEND-IMAGE} '''
            }
        }
        stage('verify') {
            steps {
                sh ''' docker ps curl http://localhost:8080/api || true '''
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
