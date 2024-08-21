pipeline {
    agent any
    options {
        timestamps()
        pipelineTriggers {
            githubPush() {
                branchFilter = 'main'
            }
        }
    }

    credentials {
        usernamePassword(credentialsId: 'dockerhub-PAT', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/churrasquito/test-app.git'
            }
        }
        stage('Build') {
            steps {
                script {
                    sh 'pip install -r requirements.txt'
                    docker.build("-t $DOCKER_USER/flask-restapi:${env.GIT_COMMIT} .")
                }
            }
        }
        stage('Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/') {
                        docker.image("$DOCKER_USER/flask-restapi:${env.GIT_COMMIT}").withCredentials([usernamePassword(credentialsId: 'dockerhub-PAT', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                            push()
                        }
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    docker.withRun("-d -p 5000:5000 $DOCKER_USER/flask-restapi:${env.GIT_COMMIT}")
                }
            }
        }
    }
}
