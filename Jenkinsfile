pipeline {
    agent any

    parameters {
        booleanParam(name: 'ENABLE_TRIVY', defaultValue: false, description: 'Enable Trivy security scan')
    }

    environment {
        IMAGE_NAME = "adi144/hello-world-demo:${BUILD_NUMBER}"
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

        stage('Trivy Scan') {
            when {
                expression { params.ENABLE_TRIVY == true }
            }
            steps {
                sh 'trivy image --severity HIGH,CRITICAL --exit-code 1 --no-progress $IMAGE_NAME'
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'e196b728-49da-413a-a925-b1ebbb09680f', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Image') {
            steps {
                sh 'docker push $IMAGE_NAME'
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker pull $IMAGE_NAME
                    docker rm -f hello-app || true
                    docker run -d --name hello-app -p 8085:80 $IMAGE_NAME
                '''
            }
        }
    }
}
