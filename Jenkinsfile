pipeline {
    agent any

    environment {
        DOCKER_USER = credentials('Docker_Hub_Id_Pwd').username
        DOCKER_PASS = credentials('Docker_Hub_Id_Pwd').password

        IMAGE_NAME = "notes-app"
        IMAGE_TAG = "v1.${BUILD_NUMBER}"   // auto versioning
        CONTAINER_NAME = "notes-app-container"
        PORT = "9092"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/Satyams-git/notes-app.git'
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
                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                '''
            }
        }

        stage('Tag & Push Image') {
            steps {
                sh '''
                echo "==== Tagging Image ===="
                docker tag $IMAGE_NAME:$IMAGE_TAG $DOCKER_USER/$IMAGE_NAME:$IMAGE_TAG

                echo "==== Pushing Image ===="
                docker push $DOCKER_USER/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Stop Old Container') {
            steps {
                sh '''
                echo "==== Stopping old container ===="
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true
                '''
            }
        }

        stage('Run New Container') {
            steps {
                sh '''
                echo "==== Running new container ===="
                docker run -d \
                --name $CONTAINER_NAME \
                -p $PORT:80 \
                -v notes-data:/data \
                $DOCKER_USER/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                echo "==== Verifying App ===="
                sleep 10
                curl -s http://15.206.100.42:$PORT | head -n 20
                '''
            }
        }
    }

    post {
        success {
            echo "Deployment Successful: http://15.206.100.42:${PORT}"
        }
        failure {
            echo "Deployment Failed! Check logs."
        }

        always {
            sh '''
            echo "==== Cleaning unused images ===="
            docker image prune -f
            '''
        }
    }
}
