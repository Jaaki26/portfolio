pipeline {
    agent {
        docker {
            image 'janakiram26/maven-docker'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        DOCKER_IMAGE = "janakiram26/about-me-website"
        DOCKER_REGISTRY = "https://index.docker.io/v1/"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
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
                sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("${DOCKER_REGISTRY}", 'docker-hub-credentials') {
                        sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Docker image ${DOCKER_IMAGE}:${IMAGE_TAG} built and pushed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Please check the logs for errors."
        }
    }
}
