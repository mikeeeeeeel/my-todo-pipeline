pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        IMAGE_NAME = "bayaras009/todo-app"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build Docker Image') {
            steps {
                sh """
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker build --no-cache -t $IMAGE_NAME:${BUILD_NUMBER} ./app
                """
            }
        }
        stage('Push to DockerHub') {
            steps {
                sh "docker push $IMAGE_NAME:${BUILD_NUMBER}"
            }
        }
        stage('Update Deployment Repo') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    sh """
                        rm -rf my-todo-deploy
                        git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/mikeeeeeeel/my-todo-deploy.git
                        cd my-todo-deploy
                        sed -i "s|image:.*|image: $IMAGE_NAME:${BUILD_NUMBER}|" ./manifests/deployment.yaml
                        git config --global user.email "ci@jenkins"
                        git config --global user.name "Jenkins CI"
                        git add ./manifests/deployment.yaml
                        git commit -m "Update deployment image to ${BUILD_NUMBER}"
                        git push https://${GIT_USER}:${GIT_TOKEN}@github.com/mikeeeeeeel/my-todo-deploy.git main
                    """
                }
            }
        }
    }
}