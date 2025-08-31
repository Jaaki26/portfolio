pipeline {
    agent {
        docker {
            image 'janakiram26/maven-docker:latest'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    // Parameters to make the pipeline flexible
    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Git branch to build')
        string(name: 'K8S_NAMESPACE', defaultValue: 'default', description: 'Kubernetes namespace')
    }

    environment {
        DOCKER_IMAGE = "janakiram26/about-me-website"
        K8S_DEPLOYMENT = "about-me"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${params.BRANCH}", url: 'https://github.com/Jaaki26/portfolio.git'
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn clean package -DskipTests -B'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build --pull -t $DOCKER_IMAGE:${BUILD_NUMBER} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub-creds', url: 'https://index.docker.io/v1/']) {
                    sh "docker push $DOCKER_IMAGE:${BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                kubectl -n ${params.K8S_NAMESPACE} set image deployment/$K8S_DEPLOYMENT $K8S_DEPLOYMENT=$DOCKER_IMAGE:${BUILD_NUMBER} --record
                kubectl -n ${params.K8S_NAMESPACE} rollout status deployment/$K8S_DEPLOYMENT
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
        always {
            cleanWs()
        }
    }
}
