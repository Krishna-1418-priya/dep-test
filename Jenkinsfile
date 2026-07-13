pipeline {
    agent any

    environment {
        // Reference the secret ID configured in Jenkins
        KUBECONFIG_PATH = credentials('minikube-config')
        // Define the deployment name as a variable to prevent mismatches
        DEPLOYMENT_NAME = 'my-app'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Repo-Testing-Account/dep-test.git'
            }
        }

        stage('Deploy to Minikube') {
            steps {
                // Set the KUBECONFIG environment variable for the duration of this block
                withEnv(['KUBECONFIG=' + env.KUBECONFIG_PATH]) {
                    echo "Deploying ${env.DEPLOYMENT_NAME} to Minikube..."
                    
                    // Apply your Kubernetes manifests
                    sh 'kubectl apply -f k8s/deployment.yaml'
                    
                    // Verify deployment rollout using the variable
                    sh "kubectl rollout status deployment/${env.DEPLOYMENT_NAME}"
                }
            }
        }
    }

    post {
        success {
            echo "Deployment of ${env.DEPLOYMENT_NAME} successful!"
        }
        failure {
            echo "Deployment failed."
        }
    }
}
