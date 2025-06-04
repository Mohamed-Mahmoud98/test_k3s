pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "mohamedmahmoud64/movie_website"
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('Website_image') {
                    sh 'docker rmi $DOCKER_IMAGE:latest || true'
                    sh 'docker build --no-cache -t $DOCKER_IMAGE:latest .'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_HUB_USER', passwordVariable: 'DOCKER_HUB_PASS')]) {
                    sh 'echo $DOCKER_HUB_PASS | docker login -u $DOCKER_HUB_USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE:latest'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // تنفيذ جميع ملفات yaml داخل مجلد k3s
                sh 'kubectl apply -f k3s/deployment.yml'
                sh 'kubectl apply -f k3s/service.yml'
                sh 'kubectl apply -f k3s/ingress.yml'
                sh 'kubectl apply -f k3s/albservice.yml'
            }
        }
    }
}
