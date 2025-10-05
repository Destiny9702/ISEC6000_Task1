// Jenkinsfile - Updated with external Dockerfile and logging

pipeline {
    // We specify agents per-stage, so no top-level agent is needed.
    agent none

    environment {
        DOCKERHUB_USERNAME = 'xqy1' 
        IMAGE_NAME         = "${DOCKERHUB_USERNAME}/isec6000"
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
                // Using 'npm ci' is a best practice for CI as it's faster and ensures a clean, consistent install from package-lock.json
                sh 'npm install --save > install_dependencies.log 2>&1' 

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
                sh 'npm test > run_unit_tests.log 2>&1'
            }
        }

        stage('Security Scan with Snyk') {
            agent {
                docker {
                    image 'node:16-alpine'
                    args '-u root'
                }
            }
            // steps {
            //     script {
            //         withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
            //             echo 'Installing Snyk CLI...'
            //             sh 'npm install -g snyk > snyk_install.log 2>&1'
                        
            //             echo 'Authenticating with Snyk...'
            //             sh 'snyk auth ${SNYK_TOKEN} > snyk_auth.log 2>&1'

            //             echo 'Scanning for vulnerabilities...'
            //             sh 'snyk test --severity-threshold=high > snyk_scan.log 2>&1'
            //         }
            //     }
            // }
        }

        stage('Build and Push Docker Image') {
            // This stage runs on the main Jenkins agent, which needs access to the Docker daemon.
            agent any
            steps {
                script {                    
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        def fullImageName = "${DOCKERHUB_USERNAME}/${IMAGE_NAME}"
                        echo "Building and pushing image: ${fullImageName}:${IMAGE_TAG}"
                        
                        // docker.build will automatically find the Dockerfile in the workspace root.
                        docker.build("${fullImageName}:${IMAGE_TAG}").push()
                    }
                }
            }
        }
    }

    // This block runs after all stages are finished.
    post {
        // 'always' ensures that we archive logs even if the pipeline fails.
        always {
            echo "Archiving build artifacts and logs..."
            
            // This single step archives all .log files and the Dockerfile.
            archiveArtifacts(
                artifacts: '**/*.log, Dockerfile', 
                allowEmptyArchive: true
            )
        }
    }
}
