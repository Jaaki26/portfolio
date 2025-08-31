pipeline {
    agent {
        docker {
            image 'maven:3.9.6-eclipse-temurin-17' // ✅ Maven with Java 17
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        DOCKER_IMAGE = "janakiram26/about-me-website"
        DOCKER_TAG = "latest"
    }

    options {
        skipStagesAfterUnstable() // Stop pipeline if a stage fails
        timestamps()             // Show timestamps in logs
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Jaaki26/project.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $DOCKER_IMAGE:$DOCKER_TAG .
                    docker images | grep $DOCKER_IMAGE
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-cred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl set image deployment/about-me-deployment about-me-container=$DOCKER_IMAGE:$DOCKER_TAG || kubectl apply -f k8s/deployment.yaml
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs.'
        }
        always {
            cleanWs() // Clean workspace after build
        }
    }
}
