pipeline {
    agent any

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
                sh """
                sed -i 's|image:.*|image: $IMAGE_NAME:${BUILD_NUMBER}|' ./manifests/deployment.yaml
                git config --global user.email "ci@jenkins"
                git config --global user.name "Jenkins CI"
                git add ./manifests/deployment.yaml
                git commit -m "Update image to version ${BUILD_NUMBER}"
                git push origin main
                """
            }
        }
    }
}
