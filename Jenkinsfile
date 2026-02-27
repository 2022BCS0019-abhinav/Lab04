pipeline {
    agent any

    environment {
        IMAGE = "abhinavbcs0019/wine-api-2022bcs0019"
        CONTAINER_NAME = "wine-api-test"
        NETWORK = "jenkins-net"
    }

    stages {

        stage('Create Network') {
            steps {
                sh "docker network create $NETWORK || true"
            }
        }

        stage('Pull Image') {
            steps {
                sh "docker pull $IMAGE"
            }
        }

        stage('Run Container') {
            steps {
                sh """
                docker run -d --network $NETWORK --name $CONTAINER_NAME $IMAGE
                """
            }
        }

        stage('Wait for Service Readiness') {
            steps {
                script {
                    timeout(time: 1, unit: 'MINUTES') {
                        waitUntil {
                            def response = sh(
                                script: "docker exec $CONTAINER_NAME curl -s -o /dev/null -w '%{http_code}' http://localhost:8000/docs || true",
                                returnStdout: true
                            ).trim()
                            return (response == "200")
                        }
                    }
                }
            }
        }

        stage('Valid Inference Test') {
            steps {
                script {
                    def response = sh(
                        script: "docker exec $CONTAINER_NAME curl -s -X POST http://localhost:8000/predict -H 'Content-Type: application/json' -d @/app/tests/valid_input.json",
                        returnStdout: true
                    ).trim()

                    echo "Valid Response: ${response}"

                    if (!response.contains("wine_quality")) {
                        error("Prediction field missing!")
                    }
                }
            }
        }

        stage('Invalid Inference Test') {
            steps {
                script {
                    def response = sh(
                        script: "docker exec $CONTAINER_NAME curl -s -o /dev/null -w '%{http_code}' -X POST http://localhost:8000/predict -H 'Content-Type: application/json' -d @/app/tests/invalid_input.json",
                        returnStdout: true
                    ).trim()

                    echo "Invalid Request HTTP Code: ${response}"

                    if (response == "200") {
                        error("Invalid request did not fail!")
                    }
                }
            }
        }

        stage('Stop Container') {
            steps {
                sh """
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true
                """
            }
        }
    }

    post {
        always {
            sh "docker stop $CONTAINER_NAME || true"
            sh "docker rm $CONTAINER_NAME || true"
        }
    }
}