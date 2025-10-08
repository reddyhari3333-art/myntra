pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = "harireddy2910/myntra" // Full image name
        IMAGE_TAG = "v1"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/reddyhari3333-art/myntra.git'
                sh 'ls -l'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                  echo "Building Docker image..."
                  docker build -t $DOCKERHUB_REPO:$IMAGE_TAG .
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred',
                                                  usernameVariable: 'DOCKER_USER',
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                      echo "Logging into DockerHub..."
                      echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                      docker push $DOCKERHUB_REPO:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                  echo "Running Myntra container locally..."

                  # Stop and remove old container
                  docker rm -f myntra-container || true

                  # Remove any old swarm service
                  docker service rm myntra || true

                  # Run new container
                  docker run -d --name myntra-container -p 8081:80 $DOCKERHUB_REPO:$IMAGE_TAG

                  echo "Myntra container is up at http://<server-ip>:8081"
                '''
            }
        }

        stage('Deploy to Docker Swarm') {
            steps {
                sh '''
                  echo "Deploying Myntra app on Swarm..."

                  # Initialize swarm (no error if already initialized)
                  docker swarm init || true

                  # Remove old service
                  docker service rm myntra || true

                  # Create new service
                  docker service create \
                    --name myntra \
                    --publish 8081:80 \
                    $DOCKERHUB_REPO:$IMAGE_TAG

                  echo "Myntra service deployed successfully!"
                '''
            }
        }

        stage('Cleanup') {
            steps {
                sh '''
                  echo "Cleaning up unused Docker resources..."
                  docker image prune -f
                  docker container prune -f
                '''
            }
        }
    }
}
