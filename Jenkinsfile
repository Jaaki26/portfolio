
pipeline {
    agent {
        docker {
            image 'maven:3.9.6-eclipse-temurin-17'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        DOCKER_IMAGE = "janakiram26/about-me-website"
        DOCKER_REGISTRY = "https://index.docker.io/v1/"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        KUBE_CONFIG = credentials('kubeconfig-credentials-id') // Ensure this matches your Jenkins credentials ID
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

        stage('Update Kubernetes Manifests') {
            steps {
                sh """
                sed -i 's|{{TAG}}|${IMAGE_TAG}|g' k8s/deployment.yaml
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-credentials-id', variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f k8s/'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'kubectl rollout status deployment/personal-website'
                sh 'kubectl get pods -l app=personal-website'
            }
        }
    }

    post {
        success {
            echo "✅ Deployment of ${DOCKER_IMAGE}:${IMAGE_TAG} completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Please check the logs for errors."
        }
    }
}
