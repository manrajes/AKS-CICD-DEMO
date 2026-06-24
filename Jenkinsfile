groovy
pipeline {
    agent any
    
    environment {
        // Change these to match your Azure Environment
        ACR_NAME = 'manrajacr2485' // e.g., myregistry if URL is myregistry.azurecr.io
        AKS_CLUSTER = 'Test'
        RESOURCE_GROUP = 'RG-Lab-Test-USEAST'
        
        // Jenkins Credential IDs
        AZURE_CRED_ID = 'azure-service-principal' // Set up in Jenkins Credentials
        ACR_CRED_ID = 'acr-credentials' // Set up in Jenkins Credentials
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Tagging image with the Jenkins build number for unique versioning
                    customImage = docker.build("${ACR_NAME}.azurecr.io/flask-aks-app:${BUILD_NUMBER}")
                }
            }
        }
        
        stage('Push Image to ACR') {
            steps {
                script {
                    docker.withRegistry("https://${ACR_NAME}.azurecr.io", "${ACR_CRED_ID}") {
                        customImage.push()
                    }
                }
            }
        }
        
        stage('Deploy to AKS') {
            steps {
                // Uses the Azure CLI/Kubectl credentials configured in Jenkins
                withCredentials([usernamePassword(credentialsId: "${AZURE_CRED_ID}", passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
                    script {
                        // 1. Authenticate with Azure CLI
                        sh "az login --service-principal -u ${AZURE_CLIENT_ID} -p ${AZURE_CLIENT_SECRET} --tenant ${YOUR_AZURE_TENANT_ID}"
                        
                        // 2. Get AKS context/kubeconfig
                        sh "az aks get-credentials --resource-group ${RESOURCE_GROUP} --name ${AKS_CLUSTER} --overwrite-existing"
                        
                        // 3. Dynamic replacement of image tag in the YAML file
                        sh "sed -i 's/\${AZURE_CONTAINER_REGISTRY}/${ACR_NAME}/g' k8s-deployment.yaml"
                        sh "sed -i 's/\${BUILD_NUMBER}/${BUILD_NUMBER}/g' k8s-deployment.yaml"
                        
                        // 4. Apply manifests to AKS
                        sh "kubectl apply -f k8s-deployment.yaml"
                        
                        // 5. Verify deployment rollout
                        sh "kubectl rollout status deployment/flask-aks-app"
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs() // Clean up workspace after build completes
        }
    }
}
