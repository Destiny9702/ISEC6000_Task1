// Jenkinsfile - Final, 100% Successful Version

pipeline {
    // We specify agents per-stage, so no top-level agent is needed.
    agent none

    environment {
        DOCKERHUB_USERNAME = 'xqy1' 
        IMAGE_NAME         = "${DOCKERHUB_USERNAME}/isec6000-nodejs-app"
        IMAGE_TAG          = "1.0.${BUILD_NUMBER}"
    }

    stages {

        stage('Install Dependencies') {
            agent {
                docker {
                    image 'node:16-alpine'
                    args '-u root'
                }
            }
            steps {
                echo 'Installing NPM dependencies using npm ci...'
                sh 'npm ci'
            }
        }

        stage('Run Unit Tests') {
            agent {
                docker {
                    image 'node:16-alpine'
                    args '-u root'
                }
            }
            steps {
                echo 'Running unit tests...'
                sh 'npm test'
            }
        }

        stage('Security Scan with Snyk') {
            agent {
                docker {
                    image 'node:16-alpine'
                    args '-u root'
                }
            }
            steps {
                script {
                    withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                        echo 'Installing Snyk CLI...'
                        sh 'npm install -g snyk'
                        echo 'Authenticating with Snyk...'
                        sh 'snyk auth ${SNYK_TOKEN}'
                        echo 'Scanning for vulnerabilities...'
                        sh 'snyk test --severity-threshold=high'
                    }
                }
            }
        }

        stage('Build and Push Docker Image') {
            agent any
            steps {
                script {
                    writeFile file: 'Dockerfile', text: """
                    FROM node:16-alpine
                    WORKDIR /app
                    COPY package*.json ./
                    RUN npm install --production
                    COPY . .
                    EXPOSE 3000
                    CMD ["node", "app.js"]
                    """

                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        echo "Building and pushing image: ${IMAGE_NAME}:${IMAGE_TAG}"
                        docker.build("${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }
    }
}
