pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        DOCKER_IMAGE = "pavansaikalyan/hotel:${BUILD_NUMBER}"
        NEXUS_URL = "http://54.172.140.97:8081"
        SONARQUBE = "sonarqube-server"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }

        stage('Checkout Code') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/kalyan3201/hotel.git'
                    ]]
                ])
            }
        }

        stage('Verify Files') {
            steps {
                sh '''
                echo "Current directory:"
                pwd

                echo "Listing all files:"
                ls -R

                echo "Checking k8s folder:"
                ls k8s
                '''
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
                    withCredentials([string(credentialsId: 'sonarqubetocken', variable: 'SONAR_TOKEN')]) {
                        sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=hotel-app \
                        -Dsonar.host.url=http://54.172.140.97:9000 \
                        -Dsonar.login=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                sh '''
                curl -u admin:admin \
                --upload-file target/restaurant-site.war \
                http://54.172.140.97:8081/repository/maven-releases/
                '''
            }
        }

        stage('Remove Old Docker Image') {
            steps {
                sh 'docker rmi -f $DOCKER_IMAGE || true'
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
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                    export KUBECONFIG=$KUBECONFIG

                    echo "Checking cluster connection..."
                    kubectl get nodes

                    echo "Verifying k8s files..."
                    ls k8s

                    echo "Applying Kubernetes manifests..."
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml

                    echo "Updating image in deployment..."
                    kubectl set image deployment/hotel-deploy hotel-container=$DOCKER_IMAGE

                    echo "Verifying rollout..."
                    kubectl rollout status deployment/hotel-deploy
                    '''
                }
            }
        }
    }
}
