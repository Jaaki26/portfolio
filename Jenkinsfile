
pipeline {
agent {
docker {
image 'maven:3.9.6-eclipse-temurin-17'
args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
}
}
environment {
DOCKER_IMAGE = "janakiram26/about-me-website"
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
            sh "docker build -t $DOCKER_IMAGE:${BUILD_NUMBER} ."  
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
            kubectl set image deployment/about-me about-me=$DOCKER_IMAGE:${BUILD_NUMBER} --record  
            kubectl rollout status deployment/about-me  
            """  
        }  
    }  
}

}. Jenkis file update it

