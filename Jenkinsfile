pipeline {
    agent any

    tools {
        // Use the names you configured in Jenkins -> Manage Jenkins -> Global Tool Configuration
        maven 'maven3'   // Replace with your Maven installation name
        jdk 'jdk17'          // Replace with your JDK installation name
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
                withSonarQubeEnv('sonarqube') { // 'sonarqube' must match Jenkins global config name
                    withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_TOKEN')]) {
                        sh """
                            mvn sonar:sonar \
                            -Dsonar.projectKey=samsung-mobile-site \
                            -Dsonar.host.url=http://52.54.241.7:9000 \
                            -Dsonar.login=$SONAR_TOKEN
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
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
