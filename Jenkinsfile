pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "pavansaikalyan/hotel:latest"
        NEXUS_URL = "http://54.172.140.97:8081"
        SONARQUBE = "sq"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/kalyan3201/hotel.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                sh '''
                curl -u admin:admin \
                --upload-file target/*.jar \
                http://54.172.140.97:8081/repository/maven-releases/
                '''
            }
        }

        stage('Remove Old Docker Image') {
            steps {
                sh '''
                docker rmi -f $DOCKER_IMAGE || true
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
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh '''
                    docker login -u $USER -p $PASS
                    docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl set image deployment/hotel-deploy hotel-container=$DOCKER_IMAGE --record
                '''
            }
        }
    }
}
