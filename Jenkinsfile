pipeline {
  agent any

  environment {
    AWS_REGION = 'ap-south-1'
    ECR_REPO = 062344139356.dkr.ecr.ap-south-5.amazonaws.com/grono-web'
  }

  triggers {
    // Automatically trigger when GitHub push event happens
    githubPush()
  }

  stages {
    stage('Checkout Code') {
      steps {
        git branch: 'main', url: 'git@github.com:Vikashguliya/grono-web.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh '''
          echo "🛠️ Building Docker image..."
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

