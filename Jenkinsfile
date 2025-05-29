pipeline {
    agent any

    tools {
        maven 'Maven3'
    }

    environment {
        IMAGE_NAME = 'najedsoumia/petclinic-app'
        TAG = 'v1'
    }

    stages {
        stage('Build Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Prepare WAR') {
            steps {
                sh 'cp target/petclinic.war .'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$TAG .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME:$TAG
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    try {
                        sh 'kubectl apply -f petclinic-deployment.yaml --validate=false'
                        sh 'kubectl apply -f petclinic-service.yaml --validate=false'
                    } catch (Exception e) {
                        echo "⚠️ Déploiement échoué : ${e.getMessage()}"
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
    }
}