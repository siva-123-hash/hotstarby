pipeline {
    agent any

    stages {
        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                  docker rmi -f hotstar:v1 || true
                  docker build -t hotstar:v1 -f Dockerfile .
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                  docker rm -f hotstar || true
                  docker run -d --name hotstar -p 8081:8080 hotstar:v1
                '''
            }
        }

        stage('Check Docker') {
            steps {
                sh 'docker ps'
            }
        }
    }
}
