pipeline {
    agent any

    stages {
        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', 
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS')]) {

                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }
    }
        environment {
            IMAGE_NAME = "mohdgayas/my_app"
            CONTAINER_NAME = "my_app"
        }

    stages {

        stage('Build docker image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
            }
        }

        stage('Login to Docker Hub') {
            steps{
                script {
                    withCredentials([usernameColonPassword(credentialsId: 'dockerhub-creds', variable: 'USERPASS')]) {
                        // The USERPASS environment variable is now available
                        // You can parse it or use it directly as needed
                        def parts = env.USERPASS.split(':')
                        def username = parts[0]
                        def password = parts[1]
    
                        sh "echo 'Logging in to Docker Hub with username: ${username}'"
                        sh "echo ${password} | docker login --username ${username} --password-stdin"
                    }
                }
            }
        }

        stage('Push image') {
            steps {
                sh ' docker push $IMAGE_NAME:$BUILD_NUMBER'
            }
        }

        stage('Run container') {
            steps {
                sh '''
                   docker stop $CONTAINER_NAME || true
                   docker rm $CONTAINER_NAME || true

                   docker run -d \
                    --name $CONTAINER_NAME \
                    -p 80:80 \
                    $IMAGE_NAME:$BUILD_NUMBER
                '''
            }
        }
    }
}
