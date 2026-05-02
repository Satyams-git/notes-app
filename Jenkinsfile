pipeline {
    agent { label "dev" }

    environment {
        DOCKER_CREDS = credentials('Docker_Hub_Id_Pwd')  

        IMAGE_NAME     = "notes-app"
        IMAGE_TAG      = "v1.${BUILD_NUMBER}"
        CONTAINER_NAME = "notes-app-container"
        PORT           = "9092"
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
                echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin
                '''
            }
        }

        stage('Tag & Push Image') {
            steps {
                sh '''
                echo "==== Tagging Image ===="
                docker tag $IMAGE_NAME:$IMAGE_TAG $DOCKER_CREDS_USR/$IMAGE_NAME:$IMAGE_TAG

                echo "==== Pushing Image ===="
                docker push $DOCKER_CREDS_USR/$IMAGE_NAME:$IMAGE_TAG
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
                $DOCKER_CREDS_USR/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                echo "==== Verifying App ===="
                sleep 10
                curl -s http://43.205.217.89:$PORT | head -n 20
                '''
            }
        }
    }

    post {
        success {
            echo "Deployment Successful: http://43.205.217.89:${PORT}"
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
