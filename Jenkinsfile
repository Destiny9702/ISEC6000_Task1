pipeline {
    agent {
        docker {
            image 'node:16-alpine'
            args '''
-u root
--network jenkins_net
-e DOCKER_HOST=tcp://docker:2376
-e DOCKER_CERT_PATH=/certs/client
-e DOCKER_TLS_VERIFY=1
-v jenkins-docker-certs:/certs/client:ro
'''
        }
    }

    environment {
        DOCKERHUB_USERNAME = 'xqy1'
        IMAGE_NAME         = "${DOCKERHUB_USERNAME}/isec6000"
        IMAGE_TAG          = "1.0.${BUILD_NUMBER}"
    }

    stages {

        stage('Prepare Build Environment') {
            steps {
                echo 'Installing Docker CLI...'
                sh 'apk add --no-cache docker-cli > prepare_environment.log 2>&1'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies...'
                sh 'npm install --save > install_dependencies.log 2>&1'
            }
        }

        stage('Run Unit Tests') {
            steps {
                echo 'Running unit tests...'
                sh 'npm test > run_unit_tests.log 2>&1'
            }
        }

        // Optional: Security Scan with Snyk
        // Uncomment if you have Snyk token configured
        // stage('Security Scan with Snyk') {
        //     steps {
        //         script {
        //             withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
        //                 sh 'npm install -g snyk'
        //                 sh 'snyk auth ${SNYK_TOKEN}'
        //                 sh 'snyk test --severity-threshold=high > security_scan.log 2>&1'
        //             }
        //         }
        //     }
        // }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${IMAGE_NAME}:${IMAGE_TAG}"
                    
                    // Build the Docker image
                    sh """
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} . > docker_build.log 2>&1
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                    """
                    
                    // Push to Docker Hub
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', 
                                                      usernameVariable: 'DOCKER_USER', 
                                                      passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                            docker push ${IMAGE_NAME}:${IMAGE_TAG} > docker_push.log 2>&1
                            docker push ${IMAGE_NAME}:latest >> docker_push.log 2>&1
                            docker logout
                        """
                    }
                    
                    echo "Successfully pushed ${IMAGE_NAME}:${IMAGE_TAG} and ${IMAGE_NAME}:latest"
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
            // Archive logs
            archiveArtifacts artifacts: '*.log', allowEmptyArchive: true
        }
        success {
            echo "Build successful! Image pushed: ${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo 'Build failed. Check logs for details.'
        }
    }
}
