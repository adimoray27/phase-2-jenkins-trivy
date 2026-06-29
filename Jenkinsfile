pipeline {
    agent any

    parameters {
        booleanParam(name: 'ENABLE_TRIVY', defaultValue: false, description: 'Enable Trivy security scan')
    }

    environment {
        IMAGE_NAME = "YOUR_DOCKERHUB_USERNAME/hello-world-demo:${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Test Website') {
            steps {
                sh '''
                    docker rm -f hello-test || true
                    docker run -d --name hello-test -p 8090:80 $IMAGE_NAME
                    sleep 5
                    curl -f http://localhost:8090
                    docker rm -f hello-test || true
                '''
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Image') {
            steps {
                sh 'docker push $IMAGE_NAME'
            }
        }
    }
}
