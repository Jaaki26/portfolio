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
        KUBE_CONFIG = credentials('kubeconfig-credentials-id') // Optional: if using kubeconfig as a secret
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
                sh "docker build -t ${DOCKER_IMAGE}:latest ."
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("${DOCKER_REGISTRY}", 'docker-hub-credentials') {
                        sh "docker push ${DOCKER_IMAGE}:latest"
                    }
                }
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
                sh 'kubectl rollout status deployment/about-me-deployment'
                sh 'kubectl get pods -l app=about-me'
            }
        }
    }

    post {
        success {
            echo '✅ Deployment completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs for details.'
        }
    }
}
