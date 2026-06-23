pipeline {
    agent any

    environment {
        // Define variables to use across the pipeline
        REGISTRY_NAME = "my-docker-registry"
        IMAGE_NAME    = "jenkins-sample-app"
        IMAGE_TAG     = "${BUILD_NUMBER}"
    }
 tools {
        nodejs 'Test1' // Name must match exactly what you set in Tools
    }

    stages {
        stage('Install Dependencies') {
            steps {
                echo 'Installing application dependencies...'
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running unit tests...'
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${IMAGE_NAME}:${IMAGE_TAG}..."
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                // Tagging it as latest for convenient deployment targets
                sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
            }
        }

        stage('Deploy / Push') {
            steps {
                echo 'Deploying application...'
                // Example placeholder for registry push or server deployment:
                // sh "docker push ${REGISTRY_NAME}/${IMAGE_NAME}:${IMAGE_TAG}"
                sh "echo 'App successfully deployed to mock environment.'"
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            // Clean up node_modules or transient build artifacts if using a persistent runner
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs above for details.'
        }
    }
}
