pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
               git branch: 'main', url:'https://github.com/kalyanpd/samsung-mobile-site.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t kalyan3599/samsung-site:v1 .'
            }
        }

	stage('Push to DockerHub') {
    		steps {
        		withCredentials([usernamePassword(credentialsId: '1234', usernameVariable: 'kalyan3599', passwordVariable: 'PASS')]) {
            			sh """
            			echo $PASS | docker login -u $USER --password-stdin
            			docker push kalyan3599/samsung-site:v1
           			 """
       				 }
   			 }		
		}

        stage('Run Container (Local Test)') {
            steps {
                sh '''
                docker rm -f samsung-site-test || true
                docker run -d -p 8081:80 --name samsung-site-test kalyan3599/samsung-site:v1
                '''
            }
        }
    }

    post {
        always {
            sh 'docker ps -a'
        }
        cleanup {
            sh 'docker rm -f samsung-site-test || true'
        }
    }
}

