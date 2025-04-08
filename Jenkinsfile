pipeline {
    agent any
    environment {
        IMAGE_NAME = 'yp3yp3/to_do_list'
        VERSION = "${BUILD_NUMBER}"
        email = 'yp3yp3@gmail.com'
    }
    stages {
        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t myapp ./app
                '''
            }
        }
        stage('Run app with Docker compose') {
            steps {
                sh '''
                    docker compose down || true
                    docker compose up -d
                '''
            }
        }
        stage('Run Tests') {
            steps {
                sh '''
                    python3 -m venv .venv
                    . .venv/bin/activate
                    pip install -r tests/requirements.txt
                    pytest ./tests
                '''
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    script {
                        docker.withRegistry('', 'docker-hub') {
                            docker.image("${IMAGE_NAME}").push("${VERSION}")
                            docker.image("${IMAGE_NAME}").push('latest')    
                        }
                    }
                   }
                }
            }
        }
        post {
             failure {
                emailext(
                    subject: "${JOB_NAME}.${BUILD_NUMBER} FAILED",
                    mimeType: 'text/html',
                    to: "$email",
                    body: "${JOB_NAME}.${BUILD_NUMBER} FAILED"
                )
            }
            success {
                emailext(
                    subject: "${JOB_NAME}.${BUILD_NUMBER} PASSED",
                    mimeType: 'text/html',
                    to: "$email",
                    body: "${JOB_NAME}.${BUILD_NUMBER} PASSED"
                )
            }
            always {
                sh '''
                    docker compose down || true
                '''
            }
        }
    }



