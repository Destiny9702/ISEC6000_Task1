pipeline {
    agent {
        docker {
            image 'node:16-alpine'
            args '''
                -u root 
                -e DOCKER_HOST=tcp://docker:2376 
                -e DOCKER_CERT_PATH=/certs/client 
                -e DOCKER_TLS_VERIFY=1 
                -v /certs/client:/certs/client:ro
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
                    
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        echo "Building and pushing image..."
                        docker.build("${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }
    }

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
