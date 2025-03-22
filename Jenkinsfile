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
            file(credentialsId: 'EC2_KEY', variable: 'EC2_KEY_PATH'),
            string(credentialsId: 'DOCKERHUB_API_KEY', variable: 'DOCKERHUB_TOKEN')
        ]) {
            sh '''
            chmod 600 "$EC2_KEY_PATH"
            ssh -o StrictHostKeyChecking=no -i "$EC2_KEY_PATH" ${EC2_USER}@${EC2_HOST} "
            # Login to Docker Hub - note we're using direct credentials here
            docker login -u ${DOCKERHUB_USER} -p ${DOCKERHUB_TOKEN}
            
            # Pull the image
            docker pull ${DOCKERHUB_USER}/salarypredapp:v1
            
            # Check if container exists before trying to stop/remove it
            if [ \$(docker ps -a -q -f name=salarypred) ]; then
                echo 'Stopping and removing existing container...'
                docker stop salarypred
                docker rm salarypred
            else
                echo 'No existing container found, skipping removal...'
            fi
            
            # Run the new container
            docker run -d -p 8501:8501 --name salarypred ${DOCKERHUB_USER}/salarypredapp:v1
            
            # Verify the container is running
            docker ps | grep salarypred
            "
            '''
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
