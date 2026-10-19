pipeline {
    agent { label 'dev' }
 
    environment {
        // Is line se 2 variables bante hain: DOCKER_CREDS_USR, DOCKER_CREDS_PSW
        DOCKER_CREDS   = credentials('Docker_Hub_Id_Pwd')
 
        IMAGE_NAME     = 'notes-app'
        IMAGE_TAG      = "v1.${BUILD_NUMBER}"
        CONTAINER_NAME = 'notes-app-container'
        PORT           = '9092'
    }
 
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Satyams-git/notes-app.git'
            }
        }
 
        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "==== Building Docker Image ===="
                    docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }
 
        stage('Docker Login') {
            steps {
                sh '''
                    echo "==== Docker Login ===="
                    echo "$DOCKER_CREDS_PSW" | docker login \
                        -u "$DOCKER_CREDS_USR" --password-stdin
                '''
            }
        }
 
        stage('Tag & Push Image') {
            steps {
                sh '''
                    echo "==== Tagging Image ===="
                    docker tag $IMAGE_NAME:$IMAGE_TAG \
                        $DOCKER_CREDS_USR/$IMAGE_NAME:$IMAGE_TAG
 
                    echo "==== Pushing Image ===="
                    docker push $DOCKER_CREDS_USR/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
 
        stage('Deploy Container') {
            steps {
                sh '''
                    echo "==== Replacing old container ===="
                    docker stop $CONTAINER_NAME || true
                    docker rm $CONTAINER_NAME || true
 
                    docker run -d --name $CONTAINER_NAME \
                        -p $PORT:80 \
                        -v notes-data:/data \
                        $DOCKER_CREDS_USR/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
 
        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "==== Verifying App ===="
                    sleep 10
                    curl -sf http://localhost:$PORT -o /dev/null
                    echo "App is UP on port $PORT"
                '''
            }
        }
    }
 
    post {
        success { echo "Deployment Successful - Tag: ${IMAGE_TAG}" }
        failure { echo 'Deployment Failed! Console Output check karo.' }
        always {
            sh 'docker logout || true'
            sh 'docker image prune -f'
        }
    }
}
