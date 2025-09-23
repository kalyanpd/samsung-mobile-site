pipeline {
    agent any

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
                sh 'docker build -t samsung-site:v1 .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                // Make sure the Jenkins credentials ID is 'dockerhub' 
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'kalyan3599', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    # Login to DockerHub using credentials
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    # Tag image for DockerHub
                    docker tag samsung-site:v1 \$DOCKER_USER/samsung-site:v1
                    # Push image
                    docker push \$DOCKER_USER/samsung-site:v1
                    """
                }
            }
        }

        stage('Run Container (Local Test)') {
            steps {
                sh '''
                # Remove existing test container if exists
                docker rm -f samsung-site-test || true
                # Run container locally for testing
                docker run -d -p 8081:80 --name samsung-site-test $DOCKER_USER/samsung-site:v1
                '''
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
            sh 'docker rm -f samsung-site-test || true'
        }
    }
}

