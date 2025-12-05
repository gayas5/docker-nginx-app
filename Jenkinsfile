pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('Gayas@123')
        IMAGE_NAME = "Gayas/my_app"
        CONTAINER_NAME = "my_app"
    }

    stages {

        stage('Build docker image') {
            steps {
                sh 'sudo docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                sh '''
                  echo $DOCKERHUB_CREDENTIALS_PSW | \
                  sudo docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                '''
            }
        }

        stage('Push image') {
            steps {
                sh 'sudo docker push $IMAGE_NAME:$BUILD_NUMBER'
            }
        }

        stage('Run container') {
            steps {
                sh '''
                  sudo docker stop $CONTAINER_NAME || true
                  sudo docker rm $CONTAINER_NAME || true

                  sudo docker run -d \
                    --name $CONTAINER_NAME \
                    -p 80:80 \
                    $IMAGE_NAME:$BUILD_NUMBER
                '''
            }
        }
    }
}
