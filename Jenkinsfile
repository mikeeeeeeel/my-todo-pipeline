pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        IMAGE_NAME = "bayaras009/todo-app"
    }
    stages {
        stage('Build Docker Image') {
            steps {
                sh """
                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                docker build -t $IMAGE_NAME:${BUILD_NUMBER} ./app
                """
            }
        }
        stage('Push to DockerHub') {
            steps {
                sh "docker push $IMAGE_NAME:${BUILD_NUMBER}"
            }
        }
        stage('Update K8s Manifests') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    sh '''
                        git config --global --add safe.directory /var/jenkins_home/workspace/todo-pipeline
                        sed -i "s|image:.*|image: bayaras009/todo-app:${BUILD_NUMBER}|" ./manifests/deployment.yaml
                        git config --global user.email "ci@jenkins"
                        git config --global user.name "Jenkins CI"
                        git add ./manifests/deployment.yaml
                        git diff --staged --quiet || git commit -m "Update deployment image to ${BUILD_NUMBER}"
                        git pull --rebase https://${GIT_USER}:${GIT_TOKEN}@github.com/mikeeeeeeel/my-todo-pipeline.git main
                        git push https://${GIT_USER}:${GIT_TOKEN}@github.com/mikeeeeeeel/my-todo-pipeline.git main
                    '''
                }
            }
        }

    }
}