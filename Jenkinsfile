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
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                        # نسخ ملف kubeconfig إلى workspace
                        cp $KUBECONFIG_FILE $WORKSPACE/kubeconfig
        
                        # تصدير متغير البيئة مع المسار الجديد
                        export KUBECONFIG=$WORKSPACE/kubeconfig
        
                        # تنفيذ أوامر kubectl
                        kubectl apply -f k3s/deployment.yml
                        kubectl apply -f k3s/service.yml
                        kubectl apply -f k3s/ingress.yml
                        kubectl apply -f k3s/albservice.yml
                    '''
                }
            }
        }

    }
}
