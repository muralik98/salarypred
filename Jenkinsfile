pipeline {
        agent any
            stages {
                stage('Clone Repository') {
                /* Cloning the repository to our workspace */
                steps {
                checkout scm
                }
           }
           stage('Build Image') {
                steps {
                sh 'sudo docker build -t salarypredapp:v1 .'
                }
           }
           stage('Run Image') {
                steps {
                sh 'sudo docker run -d -p 8501:8501 --name salarypred salarypredapp:v1'
                }
           }
           stage('Testing'){
                steps {
                    echo 'Process Complete..'

                    }
           }
    }
}
