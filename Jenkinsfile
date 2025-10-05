pipeline {
    agent none

    environment {
        DOCKERHUB_USERNAME = 'xqy1'
        IMAGE_NAME = "${DOCKERHUB_USERNAME}/isec6000"
        IMAGE_TAG = "1.0.${BUILD_NUMBER}"
    }

    stages {
        stage('Prepare Build Environment') {
            agent any
            steps {
                echo 'Installing Docker CLI...'
                sh 'apk add --no-cache docker-cli > prepare_environment.log 2>&1'
            }
        }

        stage('Install Dependencies') {
            agent {
                docker {
                    image 'node:16-alpine'
                    args '-u root'
                }
            }
            steps {
                echo 'Installing dependencies...'
                sh 'node -v && npm -v'
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
            steps {
                script {
                    withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                        echo 'Installing Snyk CLI...'
                        sh 'npm install -g snyk'
                        echo 'Authenticating with Snyk...'
                        sh 'snyk auth ${SNYK_TOKEN}'
                        echo 'Scanning for vulnerabilities...'
                        sh 'snyk test --severity-threshold=high > security_scan.log 2>&1'
                    }
                }
            }
        }

        stage('Build and Push Docker Image') {
            agent any
            steps {
                script {
                    echo "Building Docker image: ${IMAGE_NAME}:${IMAGE_TAG}"

                    sh """
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} . > docker_build.log 2>&1
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                    """

                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials',
                                                     usernameVariable: 'DOCKER_USER',
                                                     passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                            docker push ${IMAGE_NAME}:${IMAGE_TAG} > docker_push.log 2>&1
                            docker push ${IMAGE_NAME}:latest >> docker_push.log 2>&1
                        """
                    }
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: '*.log', allowEmptyArchive: true
                    echo 'Pipeline execution completed.'
                }
            }
        }
    }
}
