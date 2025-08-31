pipeline {
    agent any

    environment {
        REGISTRY = "docker.io"
        DOCKER_USER = "janakiram26"  // 🔹 Your DockerHub username
        IMAGE_NAME = "about-me-website"
        APP_VERSION = "v1.${BUILD_NUMBER}"
        KUBECONFIG_CREDENTIALS = "kubeconfig-cred" // 🔹 Add kubeconfig in Jenkins credentials
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/<your-username>/<your-repo>.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh "mvn clean package -DskipTests"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t $DOCKER_USER/$IMAGE_NAME:$APP_VERSION .
                docker tag $DOCKER_USER/$IMAGE_NAME:$APP_VERSION $DOCKER_USER/$IMAGE_NAME:latest
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKER_USER/$IMAGE_NAME:$APP_VERSION
                    docker push $DOCKER_USER/$IMAGE_NAME:latest
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIALS}", variable: 'KUBECONFIG')]) {
                    sh """
                    kubectl --kubeconfig=$KUBECONFIG apply -f k8s/deployment.yaml
                    kubectl --kubeconfig=$KUBECONFIG apply -f k8s/service.yaml
                    kubectl --kubeconfig=$KUBECONFIG apply -f k8s/ingress.yaml
                    kubectl --kubeconfig=$KUBECONFIG rollout status deployment <your-deployment-name>
                    """
                }
            }
        }
    }
}
