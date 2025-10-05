// Jenkinsfile
pipeline {
    // Specify agents per-stage for solving the issues of building image.
    agent none

    environment {
        // Define DockerHub username and image details
        DOCKERHUB_USERNAME = 'xqy1' 
        IMAGE_NAME = "${DOCKERHUB_USERNAME}/isec6000"
        IMAGE_TAG  = "Final"
    }
    
    // Stage 1: Install Node dependencies
    stages {
        stage('Install Dependencies') {
            agent {
                docker {
                    image 'node:16-alpine'    // use Node 16 as build environment
                    args '-u root'
                }
            }
            steps {
                echo 'Installing NPM dependencies using npm ci...'
                // Install project dependencies and log output
                sh 'npm install --save > install_dependencies.log 2>&1'
            }
        }
        
        // Stage 2: Run unit tests
        stage('Run Unit Tests') {
            agent {
                docker {
                    image 'node:16-alpine'    // same Node 16 image
                    args '-u root'
                }
            }
            steps {
                echo 'Running unit tests...'
                // run test commands and save logs
                sh 'npm test 2>&1 | tee run_unit_tests.log'
            }
        }
        
        // Stage 3: Security scan using Snyk
        stage('Security Scan with Snyk') {
            agent {
                docker {
                    image 'node:16-alpine'
                    args '-u root'
                }
            }
            steps {
                script {
                    // Use Jenkins credentials for Snyk authentication
                    withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                        echo 'Installing Snyk CLI...'
                        sh 'npm install -g snyk 2>&1 | tee snyk_install.log'
                        
                        echo 'Authenticating with Snyk...'
                        sh 'snyk auth ${SNYK_TOKEN} 2>&1 | tee snyk_auth.log'

                        echo 'Scanning for vulnerabilities...'
                        sh 'snyk test --severity-threshold=high 2>&1 | tee snyk_scan.log'
                    }
                }
            }
        }
        
        // Stage 4: Build & Push Docker image，use agent fix issues
        stage('Build and Push Docker Image') {
            agent {
                docker {
                    image 'docker:20.10.7'
                    args '''
                        // FIX #1: Run as root for permissions
                        -u root
                        --network jenkins_net
                        -v jenkins-docker-certs:/certs/client:ro
                        -v jenkins-data:/var/jenkins_home
                        -e DOCKER_HOST=tcp://docker:2376
                        -e DOCKER_CERT_PATH=/certs/client
                        -e DOCKER_TLS_VERIFY=1
                    '''
                }
            }
            steps {
                script {
                    echo "Building and pushing image: ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} . 2>&1 | tee docker_build.log"
                    
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        // Single quotes are correct here as we want the SHELL to expand the variables
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin 2>&1 | tee docker_login.log'
                    }
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG} 2>&1 | tee docker_push.log"
                    
                    echo "Successfully built and pushed ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
    }

    // Post and archive artifacts. 
    post {
    always {
        script {
            node {
                echo "Archiving build artifacts and logs..."
                // Save all .log files and Dockerfile as build artifacts
                archiveArtifacts(
                    artifacts: '**/*.log, Dockerfile', 
                    allowEmptyArchive: true
                    )
                }
            }
        }
    }
}
