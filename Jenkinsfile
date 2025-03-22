pipeline {
    agent any
    environment {
        DOCKERHUB_USER = 'muralik0115'
        DOCKER_IMAGE = 'muralik0115/salarypredapp:v1'
        EC2_USER = 'ubuntu'  
        EC2_HOST = '3.108.235.154'
    }
    stages {
        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }
        stage('Build Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'DOCKERHUB_API_KEY', variable: 'DOCKERHUB_TOKEN')]) {
                    sh 'echo $DOCKERHUB_TOKEN | docker login -u $DOCKERHUB_USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'EC2_SSH_KEY', keyFileVariable: 'EC2_KEY_PATH'),
                    string(credentialsId: 'DOCKERHUB_API_KEY', variable: 'DOCKERHUB_TOKEN')
                ]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no -i $EC2_KEY_PATH $EC2_USER@$EC2_HOST << 'EOF'
                    sudo systemctl start docker  # Ensure Docker is running
                    echo $DOCKERHUB_TOKEN | docker login -u $DOCKERHUB_USER --password-stdin
                    docker pull $DOCKER_IMAGE
                    docker stop salarypred || true
                    docker rm salarypred || true
                    docker run -d -p 8501:8501 --name salarypred $DOCKER_IMAGE
                    EOF
                    """
                }
            }
        }
        stage('Testing') {
            steps {
                echo 'Deployment complete on EC2'
            }
        }
    }
}
