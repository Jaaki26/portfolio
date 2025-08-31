pipeline {
    agent {
        docker {
            image 'maven:3.9.6-eclipse-temurin-17'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        DOCKER_IMAGE = "janakiram26/about-me-website"
        KUBE_DEPLOYMENT = "personal-website"
        CONTAINER_NAME = "about-me"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Jaaki26/portfolio.git'
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t $DOCKER_IMAGE:${BUILD_NUMBER} .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub-creds', url: 'https://index.docker.io/v1/']) {
                    sh """
                        docker push $DOCKER_IMAGE:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    # Update Deployment image
                    kubectl set image deployment/$KUBE_DEPLOYMENT $CONTAINER_NAME=$DOCKER_IMAGE:${BUILD_NUMBER} --record
                    
                    # Wait for rollout to complete
                    kubectl rollout status deployment/$KUBE_DEPLOYMENT
                """
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful: ${DOCKER_IMAGE}:${BUILD_NUMBER}"
        }
        failure {
            echo "❌ Pipeline failed. Check logs for errors."
        }
    }
}
