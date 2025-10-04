// Final Jenkinsfile for testing (Snyk stage temporarily removed)
pipeline {
    // Defines the agent for the entire pipeline.
    // It runs in a Node.js container, as root, with certificates mounted for DinD communication.
    agent {
        docker {
            image 'node:16-alpine'
            args '-u root -v jenkins-docker-certs:/certs/client:ro' 
        }
    }

    // Defines environment variables available to all stages.
    environment {
        DOCKERHUB_USERNAME = 'xqy1' 
        IMAGE_NAME         = "${DOCKERHUB_USERNAME}/isec6000-nodejs-app" // I've used the name from your Docker Hub repo for consistency
        IMAGE_TAG          = "1.0.${BUILD_NUMBER}" // Using BUILD_NUMBER makes each tag unique
    }

    stages {
        // Stage 1: Prepares the agent by installing the Docker CLI inside it.
        stage('Prepare Build Environment') {
            steps {
                echo 'Installing Docker CLI...'
                sh 'apk add --no-cache docker-cli'
            }
        }

        // Stage 2: Installs application dependencies using npm ci for consistency.
        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies...'
                sh 'npm ci'
            }
        }

        // Stage 3: Runs unit tests.
        stage('Run Unit Tests') {
            steps {
                echo 'Running unit tests...'
                sh 'npm test'
            }
        }

        // Stage 4: Builds and pushes the Docker image.
        stage('Build and Push Docker Image') {
            steps {
                // withEnv forcefully injects the necessary environment variables
                // to ensure the agent's shell can connect to the DinD container.
                withEnv([
                    "DOCKER_HOST=tcp://docker:2376",
                    "DOCKER_CERT_PATH=/certs/client",
                    "DOCKER_TLS_VERIFY=1"
                ]) {
                    script {
                        // Securely logs into Docker Hub using stored credentials.
                        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                            echo "Building and pushing image: ${IMAGE_NAME}:${IMAGE_TAG}"
                            
                            // Use sh steps for reliable execution in the configured shell environment.
                            sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                            sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                        }
                    }
                }
            }
        }
    }

    // Post-build actions that run regardless of pipeline status.
    post {
        always {
            echo "Pipeline finished. Archiving artifacts..."
            // Cleans up by archiving any log files created.
            archiveArtifacts(
                artifacts: '**/*.log', 
                allowEmptyArchive: true
            )
        }
    }
}
