// Jenkinsfile

// The 'pipeline' block is the root for all Declarative Pipelines.
pipeline {
    // The 'agent' section specifies where the entire Pipeline will execute.
    // Requirement 1.b.i: Use the Node 16 Docker image as the build agent.
    agent {
        docker {
            image 'node:16-alpine' // Use a lightweight, Alpine-based Node 16 image.
            args '-u root' // Run as the root user inside the container to avoid permission issues with npm and Docker commands.
        }
    }

    // The 'environment' block defines environment variables that will be available to all stages.
    environment {
        //Docker Hub configuration.
        DOCKERHUB_USERNAME = 'xqy1' 
        IMAGE_NAME         = "${DOCKERHUB_USERNAME}/isec6000-nodejs-app"
        // Use Jenkins' built-in BUILD_NUMBER variable to generate a unique tag for each build.
        IMAGE_TAG          = "1.0.${BUILD_NUMBER}"
    }

    // The 'stages' block contains all the execution stages of the pipeline.
    stages {

        // Stage 1: Install Project Dependencies
        // Requirement 1.b.ii (part 1): Install dependencies.
        stage('Install Dependencies') {
            steps {
                echo 'Installing NPM dependencies using npm ci...'
                // Use 'npm ci' instead of 'npm install'. The 'ci' command performs a clean install based on the 
                // package-lock.json file, which is faster and ensures consistent builds in a CI environment. This is a best practice.
                sh 'npm ci'
            }
        }

        // Stage 2: Run Unit Tests
        // Requirement 1.b.ii (part 2): Run unit tests.
        stage('Run Unit Tests') {
            steps {
                echo 'Running unit tests...'
                sh 'npm test'
            }
        }

        // Stage 3: Security Scan with Snyk
        // Requirement 2.a & 2.b: Integrate a vulnerability scanner and fail the build if high/critical issues are detected.
        stage('Security Scan with Snyk') {
            steps {
                script {
                    // The withCredentials block securely loads the Snyk token from the Jenkins Credentials store.
                    withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                        echo 'Installing Snyk CLI...'
                        sh 'npm install -g snyk' // Globally install the Snyk CLI tool in the build environment.

                        echo 'Authenticating with Snyk...'
                        sh 'snyk auth ${SNYK_TOKEN}' // Authenticate using the token.

                        echo 'Scanning for vulnerabilities...'
                        // Run 'snyk test' with '--severity-threshold=high'.
                        // If Snyk finds any vulnerabilities of 'high' or 'critical' severity, this command will exit with a non-zero status code,
                        // which automatically fails the current stage and stops the pipeline.
                        sh 'snyk test --severity-threshold=high'
                    }
                }
            }
        }

        // Stage 4: Build and Push Docker Image
        // Requirement 1.b.ii (parts 3 & 4): Build a Docker image and push it to a registry.
        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Step 1: Dynamically create a Dockerfile in the workspace.
                    // The 'docker build .' command needs this file to exist.
                    writeFile file: 'Dockerfile', text: """
                    FROM node:16-alpine
                    WORKDIR /app
                    COPY package*.json ./
                    RUN npm install --production
                    COPY . .
                    EXPOSE 3000
                    CMD ["node", "app.js"]
                    """
    
                    echo "Building Docker image: ${IMAGE_NAME}:${IMAGE_TAG}"
                    // Step 2: Use the idiomatic docker.build() command provided by the Docker Pipeline plugin.
                    // This command runs in the correct context (the Jenkins controller), not inside the node agent.
                    def customImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
    
                    // Step 3: Use withCredentials to securely push the image.
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        echo "Pushing Docker image to Docker Hub..."
                        // The .push() method is called on the image object returned by docker.build().
                        // This also runs in the correct context.
                        customImage.push()
                    }
                }
            }
        }
    }

    // The 'post' block defines actions that will be run at the end of the Pipeline's execution.
    post {
        // 'always' means the steps inside will execute regardless of the Pipeline's success, failure, or abortion.
        always {
            echo 'Pipeline finished. Cleaning up workspace...'
            // cleanWs() is a built-in function that deletes the workspace files, which helps save disk space on the Jenkins server.
            cleanWs()
        }
    }
}
