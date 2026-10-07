pipeline {
    agent any

    stages {

        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        stage('JUnit Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Gitleaks') {
            steps {
                sh 'gitleaks detect --source . --no-banner'
            }
        }

        stage('Docker Image Build') {
            steps {
                sh 'docker build -t hackathon-app:jenkins .'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL hackathon-app:jenkins'
            }
        }

        stage('Docker Run') {
            steps {
                sh 'docker rm -f novabank-jenkins || true'
                sh 'docker run -d --name novabank-jenkins -p 8082:8082 hackathon-app:jenkins'
                sh 'sleep 10'
                sh 'curl -f http://localhost:8082/'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'target/surefire-reports/*.xml', allowEmptyArchive: true
        }
    }
}
