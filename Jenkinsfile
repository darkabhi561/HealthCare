pipeline {
     agent { label "Dev" }
     
     environment {
         NET = "health-care"
         DATABASE_CONTAINER = "health-contain"
         BACKEND_CONTAINER = "health-backend"
         FRONTEND_CONTAINER = "health-front"
         BACKEND_IMAGE = "healthcare-backend:${BUILD_NUMBER}"
         FRONTEND_PORT = "3000"
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
                  -v "$WORKSPACE/db":/docker-entrypoint-initdb.d \
                  postgres:16
                '''
            }
        }
        
         stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'docker build -t $BACKEND_IMAGE .'
                }
            }
        }
        
       stage('Run Backend') {
            steps {
                sh '''
               docker run -d --name $BACKEND_CONTAINER \
                 --network $NET \
                -p 5000:5000 \
                -e DATABASE_HOST=$DATABASE_CONTAINER \
                -e DATABASE_USER=qbuser \
                -e DATABASE_PASSWORD=$DATABASE_PASSWORD \
               -e DATABASE_NAME=$DATABASE_NAME \
                $BACKEND_IMAGE

                '''
            }
        }
      
       stage('Run Frontend') {
            steps {
                sh '''
                docker run -d --name $FRONTEND_CONTAINER \
                  --network $NET \
                  -p $FRONTEND_PORT:80 \
                  -v "$WORKSPACE/frontend/static/index.html":/usr/share/nginx/html/index.html \
                  nginx:latest
                '''
            }
        }
        
        
         stage('Verify') {
            steps {
                sh '''
                sleep 6
                curl -f http://13.233.93.69:5000 || (echo "App not responding" && exit 1)
                '''
            }
        }

     }
      
       post {
        success {
            echo "Deployment SUCCESS"
        }
        failure {
            echo "❌ Deployment FAILED. Debug with docker logs."
            sh 'docker ps -a'
        }
    }
        
     
    
    
}
