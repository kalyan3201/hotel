pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = "dockerhub"
        IMAGE_NAME = "hotel1"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/kalyan3201/hotel.git'
            }
        }
        stage('creating-artifact') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

      stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker tag hotelimg:latest $DOCKER_USER/hotelimg:latest
                        docker push $DOCKER_USER/hotstarimg:latest
                        docker logout
                    '''
                }
            }
        }
        stage('K8s Deployment') {
         steps {
            withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                sh '''
                export KUBECONFIG=$KUBECONFIG
                kubectl get nodes
                kubectl apply -f hotel.yml
                '''
                }
            }
        }
     }

    post {

        always {
            echo "Pipeline execution completed"
        }

        success {
            echo "Build SUCCESS "
            mail to: 'pavansaikalyan/hotelimg:latest',
                 subject: "Jenkins SUCCESS: ${env.JOB_NAME}",
                 body: "Build succeeded: ${env.BUILD_URL}"
        }

        failure {
            echo "Build FAILED "
            mail to: 'pavansaikalyan/hotelimg:latest',
                 subject: "Jenkins FAILURE: ${env.JOB_NAME}",
                 body: "Build failed: ${env.BUILD_URL}"
        }

        unstable {
            echo "Build UNSTABLE "
        }
    }
}
