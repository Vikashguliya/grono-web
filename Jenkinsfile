pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'              // ✅ EKS aur ECR ka region yahan set karo
        ECR_REPO = '062344139356.dkr.ecr.ap-south-1.amazonaws.com/grono-web' // ✅ Example repo URL
    }

    stages {
        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "🛠️  Building Docker image..."
                    docker build -t ${ECR_REPO}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                    echo "🚀 Logging in to ECR..."
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO%/*}
                    echo "📦 Pushing Docker image..."
                    docker push ${ECR_REPO}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    echo "☸️ Deploying to EKS..."
                    aws eks update-kubeconfig --region ${AWS_REGION} --name jenkins-eks-cluster   # ✅ Add this line
                    kubectl set image deployment/grono-web-deploy grono-web=${ECR_REPO}:${BUILD_NUMBER} --namespace default || true
                    kubectl rollout status deployment/grono-web-deploy
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful! Visit your LoadBalancer URL."
        }
        failure {
            echo "❌ Build failed. Check console logs for errors."
        }
    }
}

