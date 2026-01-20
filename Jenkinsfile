pipeline {
    agent any

    tools {
        maven 'maven3.9.12'
        jdk 'java17'
    }

    environment {
        MAVEN_REPO_URL = 'http://127.0.0.1:8081/repository/maven-releases/'
        DOCKER_REGISTRY = '127.0.0.1:8082/docker-releases'
        DOCKER_IMAGE = 'country-chicken-backend'
        DOCKER_TAG = '1.0.0'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/djroy2104/country-chicken-backend.git'
            }
        }

        stage('Build JAR') {
            steps {
                sh "mvn clean package -DskipTests"
            }
        }

        stage('Upload to Nexus Maven') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-credentials', 
                    usernameVariable: 'NEXUS_USER', 
                    passwordVariable: 'NEXUS_PASS')]) {
                    sh """
                        mvn deploy:deploy-file \
                          -DgroupId=com.countrychicken \
                          -DartifactId=country-chicken-backend \
                          -Dversion=${DOCKER_TAG} \
                          -Dpackaging=jar \
                          -Dfile=target/country-chicken-backend-${DOCKER_TAG}.jar \
                          -DrepositoryId=maven-releases \
                          -Durl=${MAVEN_REPO_URL} \
                          -DretryFailedDeploymentCount=3 \
                          -Dusername=$NEXUS_USER \
                          -Dpassword=$NEXUS_PASS
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG} .
                    docker tag ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', 
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo $DOCKER_PASS | docker login ${DOCKER_REGISTRY} -u $DOCKER_USER --password-stdin
                        docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest
                    """
                }
            }
        }
    }

    post {
        always {
            sh 'docker system prune -f'
            cleanWs()
        }
    }
}

