pipeline {
    agent any

    environment {
        IMAGE_NAME = "notes-app:new"
        CONTAINER_NAME = "notes-app-container"
        PORT = "9092"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Satyams-git/notes-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "==== Building Docker Image ===="
                docker build -t $IMAGE_NAME .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'Docker_Hub_Id_Pwd',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "==== Logging in to Docker Hub ===="
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                    echo "==== Tagging Image ===="
                    docker tag $IMAGE_NAME $DOCKER_USER/$IMAGE_NAME:NEW

                    echo "==== Pushing Image to Docker Hub ===="
                    docker push $DOCKER_USER/$IMAGE_NAME:NEW
                    '''
                }
            }
        }


        stage('Stop Old Container') {
            steps {
                sh '''
                echo "==== Stopping old container (if any) ===="
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true
                '''
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                echo "==== Running new container ===="
                docker run -d --name $CONTAINER_NAME -p $PORT:80 -v notes-data:/data $IMAGE_NAME
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                echo "==== Checking app response ===="
                sleep 5
                curl -s http://15.206.100.42:$PORT | head -n 20
                '''
            }
        }
    }

    post {
        success {
            echo "Notes App deployed successfully: http://15.206.100.42:${PORT}"
        }
        failure {
            echo "Build or deploy failed. Check logs."
        }
    }
}
