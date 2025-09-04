pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'siva0927'        // 🔹 your DockerHub username
        IMAGE_NAME      = 'myapp'
        IMAGE_TAG       = 'v1'
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'siva0927-dockerhub',  // 🔹 Jenkins credential ID
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "⚡ Logging into DockerHub..."
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                    echo "⚡ Tagging image..."
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}

                    echo "⚡ Pushing image..."
                    docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Run Container (Local Test)') {
            steps {
                script {
                    sh 'docker rm -f myapp-cont || true'
                    sh 'docker run -d --name myapp-cont -p 8082:8080 ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}'

                    // ✅ Show container logs in Jenkins console
                    sh 'sleep 5 && docker logs myapp-cont'
                }
            }
        }

        stage('Deploy to Docker Swarm') {
            steps {
                script {
                        // Check if swarm is active, otherwise initialize
            def swarmCheck = sh(script: "docker info --format '{{.Swarm.LocalNodeState}}'", returnStdout: true).trim()
            if (swarmCheck != "active") {
                echo "Docker Swarm not initialized. Initializing now..."
                sh 'docker swarm init || true'
            }

            // Remove old service if exists
            sh 'docker service rm myapp-service || true'

            // Deploy new service
            sh 'docker service create --name myapp-service --publish 8082:8080 --replicas 5 siva0927/myapp:v1'
                }
            }
        }
    }
}
