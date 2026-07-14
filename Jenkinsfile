pipeline {
    agent any
    
    environment {
        IMAGE_NAME = "testingacountwork/my-app"
        TAG = "${BUILD_NUMBER}"
        IMAGE_WITH_TAG = "${IMAGE_NAME}:${TAG}"
        // Ensure this ID matches your Jenkins Credentials
        DOCKER_HUB_CREDS = 'dockerhub-credentials-id' 
    }

    stages {
        stage('Verify Files') {
    steps {
        git branch: 'main', url: 'https://github.com/Krishna-1418-priya/dep-test.git'
    }
}
        stage('Build & Push') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'dockerhub-credentials-id',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )]) {

            sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker build -t ${IMAGE_WITH_TAG} .
            docker push ${IMAGE_WITH_TAG}
            docker logout
            '''
        }
    }
}

        stage('Deploy') {
    steps {
        withKubeConfig([credentialsId: 'minikube-config']) {
            script {
                // Debug: list contents of k8s directory
                sh 'ls -la k8s/'
                
                def templatePath = 'k8s/deployment.template.yaml'
                
                if (fileExists(templatePath)) {
                    sh "sed 's|IMAGE_PLACEHOLDER|${IMAGE_WITH_TAG}|g' ${templatePath} > k8s/deployment.yaml"
                    sh "kubectl apply -f k8s/deployment.yaml"
                    sh "kubectl rollout restart deployment/my-app"
                    sh "kubectl rollout status deployment/my-app"
                } else {
                    error "Could not find ${templatePath}. Please check your GitHub repository."
                }
            }
        }
    }
}
    }
    
    post {
        always {
            sh "rm -f k8s/deployment.yaml"
        }
    }
}
