pipeline {
    agent any

    tools {
        jdk 'jdk17'          // JDK 17 installed in Jenkins
        maven 'maven3'       // Maven 3 installed in Jenkins
    }

    environment {
        IMAGE_NAME = "samsung-site"
        IMAGE_TAG = "v1"
        TEST_CONTAINER_NAME = "samsung-site-test"

        // SonarQube (configured in Jenkins "Manage Jenkins > Configure System")
        SONARQUBE_ENV = 'sonarqube'   // The name you gave your SonarQube server
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

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} \$DOCKER_USER/${IMAGE_NAME}:${IMAGE_TAG}
                        docker push \$DOCKER_USER/${IMAGE_NAME}:${IMAGE_TAG}
                        docker logout
                    """
                }
            }
        }

        /*
        stage('Run Container (Local Test)') {
            steps {
                sh """
                    docker rm -f ${TEST_CONTAINER_NAME} || true
                    docker run -d -p 8081:80 --name ${TEST_CONTAINER_NAME} \$DOCKER_USER/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
        */
    }

    post {
        always {
            echo 'Pipeline completed.'
            sh 'docker ps -a'
        }
        cleanup {
            sh "docker rm -f ${TEST_CONTAINER_NAME} || true"
        }
        failure {
            echo 'Pipeline failed ❌'
        }
        success {
            echo 'Pipeline succeeded ✅'
        }
    }
}
