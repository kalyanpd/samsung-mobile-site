pipeline {
    agent any

    environment {
        IMAGE_NAME = "samsung-site"
        IMAGE_TAG = "v1"
        TEST_CONTAINER_NAME = "samsung-site-test"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/kalyanpd/samsung-mobile-site.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                // Use Jenkins credentials ID 'dockerhub' (replace with your ID)
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    # Login to DockerHub using Jenkins credentials
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    # Tag Docker image for DockerHub
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} \$DOCKER_USER/${IMAGE_NAME}:${IMAGE_TAG}
                    # Push Docker image to DockerHub
                    docker push \$DOCKER_USER/${IMAGE_NAME}:${IMAGE_TAG}
                    docker logout
                    """
                }
            }
        }

        stage('Run Container (Local Test)') {
            steps {
                sh """
                # Remove existing test container if exists
                docker rm -f ${TEST_CONTAINER_NAME} || true
                # Run container locally for testing
                docker run -d -p 8081:80 --name ${TEST_CONTAINER_NAME} \$DOCKER_USER/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
    }

    post {
        always {
            // Show all containers
            sh 'docker ps -a'
        }
        cleanup {
            // Remove test container after pipeline
            sh 'docker rm -f ${TEST_CONTAINER_NAME} || true'
        }
    }
}
