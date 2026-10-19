pipeline {
    agent { label 'dev' }
 
    environment {
        DOCKER_CREDS   = credentials('Docker_Hub_Id_Pwd')  // _USR, _PSW
 
        IMAGE_NAME     = 'notes-app'
        IMAGE_TAG      = "v1.${BUILD_NUMBER}"
        CONTAINER_NAME = 'notes-app-container'
        PORT           = '9092'
        HOST_IP        = '13.250.117.160'
        EMAIL_TO       = 'satyam.du.ae@gmail.com'
    }
 
    stages {
        stage('Checkout') {
            steps {
                checkout scm            // Jenkinsfile wali repo hi checkout hogi
            }
        }
 
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }
 
        stage('Push to Docker Hub') {
            steps {
                sh '''
                    echo "$DOCKER_CREDS_PSW" | docker login \
                        -u "$DOCKER_CREDS_USR" --password-stdin
                    docker tag $IMAGE_NAME:$IMAGE_TAG \
                        $DOCKER_CREDS_USR/$IMAGE_NAME:$IMAGE_TAG
                    docker push $DOCKER_CREDS_USR/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
 
        stage('Deploy') {
            steps {
                sh '''
                    docker stop $CONTAINER_NAME || true
                    docker rm   $CONTAINER_NAME || true
 
                    docker run -d --name $CONTAINER_NAME \
                        -p $PORT:80 \
                        -v notes-data:/data \
                        $DOCKER_CREDS_USR/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
 
        stage('Verify') {
            steps {
                sh '''
                    sleep 10
                    curl -sf http://localhost:$PORT -o /dev/null
                    echo "App is UP on port $PORT"
                '''
            }
        }
    }
 
    post {
        success {
            echo "App deployed at: http://${HOST_IP}:${PORT}"
            emailext(
                to: "${EMAIL_TO}",
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                mimeType: 'text/plain',
                body: """Build successful!
 
Job       : ${env.JOB_NAME}
Build     : #${env.BUILD_NUMBER}
Image Tag : ${IMAGE_TAG}
App URL   : http://${HOST_IP}:${PORT}
Logs      : ${env.BUILD_URL}console
"""
            )
        }
        failure {
            emailext(
                to: "${EMAIL_TO}",
                subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                mimeType: 'text/plain',
                body: """Build failed!
 
Job   : ${env.JOB_NAME}
Build : #${env.BUILD_NUMBER}
Logs  : ${env.BUILD_URL}console
"""
            )
        }
        always {
            sh 'docker logout || true'
            sh 'docker image prune -f'
        }
    }
}
