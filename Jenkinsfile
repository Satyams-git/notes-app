pipeline {

    agent any

    environment {
        DOCKER_IMAGE = "satyamsri/notes-app"
        SONAR_SCANNER = tool 'sonar-scanner'
        AWS_REGION = "ap-south-1"
        CLUSTER_NAME = "notes-eks-cluster"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Satyams-git/notes-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    echo "Building Docker Image..."
                    docker build -t ${DOCKER_IMAGE}:latest .
                """
            }
        }

        stage('Trivy Scan') {
            steps {
                sh """
                    echo "Running Trivy Vulnerability Scan..."
                    trivy image --exit-code 0 --severity HIGH,CRITICAL ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh '''
                            echo "Running SonarScanner..."
                            ${SONAR_SCANNER}/bin/sonar-scanner \
                              -Dsonar.projectKey=notes-app \
                              -Dsonar.sources=. \
                              -Dsonar.host.url=$SONAR_HOST_URL \
                              -Dsonar.token=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Wait for Sonar Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false   // 🔥 ALLOW PIPELINE TO CONTINUE
                }
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker tag satyamsri/notes-app:latest satyamsri/notes-app:${BUILD_NUMBER}

                        docker push satyamsri/notes-app:${BUILD_NUMBER}
                        docker push satyamsri/notes-app:latest
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'aws-creds',
                    usernameVariable: 'AWS_ACCESS_KEY_ID',
                    passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    sh '''
                        echo "Configuring AWS CLI..."
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set default.region ap-south-1

                        echo "Updating kubeconfig..."
                        aws eks update-kubeconfig --name notes-eks-cluster --region ap-south-1

                        echo "Applying Kubernetes manifests..."
                        kubectl apply -f k8s/

                        echo "Updating deployment image..."
                        kubectl set image deployment/notes-app notes-app=satyamsri/notes-app:${BUILD_NUMBER} -n default

                        echo "Waiting for rollout..."
                        kubectl rollout status deployment/notes-app -n default
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "Verifying Kubernetes Resources..."
                    kubectl get pods -o wide -n default
                    kubectl get svc -o wide -n default
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
