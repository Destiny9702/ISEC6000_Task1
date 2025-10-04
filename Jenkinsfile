pipeline {
        // Defines Node 16 Docker container as the agent for the entire pipeline.
    agent {
        docker {
            image 'node:16-alpine'
            args '-u root -v jenkins-docker-certs:/certs/client:ro' 
        }
    }

    environment {
        DOCKERHUB_USERNAME = 'xqy1' 
        IMAGE_NAME         = "${DOCKERHUB_USERNAME}/isec6000"
        IMAGE_TAG          = "Final"
         //  Tell the Docker CLI in the agent container how to find the DinD service over the network.
        DOCKER_HOST        = 'tcp://docker:2376'
        DOCKER_CERT_PATH   = '/certs/client'
        DOCKER_TLS_VERIFY  = '1'
    }

    stages {
        // Prepares the agent by installing the Docker CLI inside it.
        stage('Prepare Build Environment') {
            steps {
                echo 'Installing Docker CLI...'
                // Generate the log file for this stage
                sh 'apk add --no-cache docker-cli > prepare_environment.log 2>&1'
            }
        }
        // Installs application dependencies and logs the output.
        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies...'
                // Generate the log file for this stage
                sh 'npm install --save > install_dependencies.log 2>&1'
            }
        }
        // Runs unit tests and logs the output.
        stage('Run Unit Tests') {
            steps {
                echo 'Running unit tests...'
                // Generate the log file for this stage
                sh 'npm test > run_unit_tests.log 2>&1'
            }
        }
        // Performs a security vulnerability scan and logs the output.
        stage('Security Scan with Snyk') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                        sh 'npm install -g snyk'
                        sh 'snyk auth ${SNYK_TOKEN}'
                        // Generate the log file for the final scan command
                        sh 'snyk test --severity-threshold=high > security_scan.log 2>&1'
                    }
                }
            }
        }
        // Builds and pushes the Docker image from within the Node.js agent.
       stage('Build and Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        echo "Building image via sh step..."
                        sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                        echo "Pushing image via sh step..."
                        sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                }
            }
       }
    }

    // To archive build artifacts after all stages pass.
    post {
        always {
            echo "Archiving build artifacts and logs..."
            archiveArtifacts(
                artifacts: '**/*.log, Dockerfile', 
                allowEmptyArchive: true
            )
        }
    }
}
