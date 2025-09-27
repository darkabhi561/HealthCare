pipeline {
     agent any
     
     environment {
         NET = "health-care"
         DATABASE_CONTAINER = "health-contain"
         BACKEND_CONTAINER = "health-backend"
         FRONTEND_CONTAINER = "health-front"
         BACKEND_IMAGE = "healthcare-backend:${BUILD_NUMBER}"
         FRONTED_PORT = "3000"
         DATABASE_PASSWORD= "qbpass"
         DATABASE_NAME= "qbhealth"
     }
     
     stages {
         stage('Checkout') {
            steps {
                echo "Checking out GitHub repo..."
                checkout scm
            }
        }
        
        stage('Prepare Environment') {
            steps {
                sh '''
                   docker network inspect $NET >/dev/null 2>&1 || docker network create $NET
                   docker rm -f $DATABASE_CONTAINER $BACKEND_CONTAINER $FRONTEND_CONTAINER >/dev/null 2>&1 || true
                '''
            }
        }
        
        stage('start database') {
            steps {
                sh'''
                   docker run -d --name $DATABASE_CONTAINER \
                  --network $NET \
                  -e POSTGRES_PASSWORD=$DATABASE_PASSWORD \
                  -e POSTGRES_DB=$DATABASE_NAME \
                  -v health-vol:/var/lib/postgresql/data \
                  -v "$WORKSPACE/db:/docker-entrypoint-initdb.d \
                  postgres:16
                '''
            }
        }
        
        
     }
    
    
}
