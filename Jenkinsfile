pipeline {
    agent any

    tools {
        maven 'maven3'   // your Jenkins Maven tool name
        jdk 'jdk17'      // your Jenkins JDK tool name
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/kalyanpd/samsung-mobile-site.git',
                    credentialsId: 'kalyan'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {  // must match Jenkins global SonarQube server config
                    withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_TOKEN')]) {
                        sh """
                            mvn sonar:sonar \
                            -Dsonar.projectKey=samsung-mobile-site \
                            -Dsonar.host.url=http://52.54.241.71:9000 \
                            -Dsonar.login=$SONAR_TOKEN
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 1, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ SonarQube Analysis & Quality Gate passed!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
