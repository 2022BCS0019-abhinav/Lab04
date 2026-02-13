pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "abhinavbcs0019/wine-api-2022bcs0019"
        DOCKER_CREDENTIALS = "dockerhub-credentials"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/2022BCS0019-abhinav/Lab04.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install -r requirements.txt
                '''
            }
        }

        stage('Train Model') {
            steps {
                sh '''
                . venv/bin/activate
                python scripts/train.py
                '''
            }
        }

        stage('Print Metrics with Name and Roll No') {
            steps {
                sh '''
                echo "Student Name: Abhinav Bhagwat"
                echo "Roll Number: 2022BCS0019"
                cat metrics.json
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS, usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push $DOCKER_IMAGE
                    '''
                }
            }
        }
    }
}
