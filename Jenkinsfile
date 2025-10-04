pipeline {
    // Requirement: Use a single Node 16 Docker image as the build agent for the entire pipeline.
    agent {
        docker {
            image 'node:16-alpine'
            args '-u root -v /var/run/docker.sock:/var/run/docker.sock' // Mount Docker socket into the agent
        }
    }

    environment {
        DOCKERHUB_USERNAME = 'xqy1' 
        IMAGE_NAME         = "${DOCKERHUB_USERNAME}/isec6000"
        IMAGE_TAG          = "1.0.${BUILD_NUMBER}"
    }

    stages {

        // This stage prepares the build agent by installing the Docker CLI.
        stage('Prepare Build Environment') {
            steps {
                echo 'Installing Docker CLI inside the agent...'
                sh 'apk add --no-cache docker-cli'
            }
        }

        // This stage installs dependencies as requested.
        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies using npm install...'
                sh 'npm install --save'
            }
        }

        // This stage runs unit tests.
        stage('Run Unit Tests') {
            steps {
                echo 'Running unit tests...'
                sh 'npm test'
            }
        }

        // This stage performs the security scan.
        stage('Security Scan with Snyk') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                        sh 'npm install -g snyk'
                        sh 'snyk auth ${SNYK_TOKEN}'
                        sh 'snyk test --severity-threshold=high'
                    }
                }
            }
        }

        // This stage builds and pushes the image. 
        stage('Build and Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        echo "Building and pushing image: ${IMAGE_NAME}:${IMAGE_TAG}"
                        docker.build("${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline successful. Archiving the Dockerfile..."
            archiveArtifacts artifacts: 'Dockerfile', allowEmptyArchive: true
        }
    }
}
