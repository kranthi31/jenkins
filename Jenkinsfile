pipeline {
  agent any

  environment {
    AWS_REGION   = "us-west-2"
    AWS_ACCOUNT  = "548889528327"
    ECR_REPO     = "rails-app"
    ECR_REGISTRY = "${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    IMAGE_TAG    = "${BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t rails-app:${BUILD_NUMBER} .'
      }
    }

    stage('Login to ECR') {
      steps {
        sh '''
          aws ecr get-login-password --region us-west-2 | \
          docker login --username AWS --password-stdin 548889528327.dkr.ecr.us-west-2.amazonaws.com
        '''
      }
    }

    stage('Tag & Push') {
      steps {
        sh '''
          docker tag rails-app:${BUILD_NUMBER} \
            548889528327.dkr.ecr.us-west-2.amazonaws.com/rails-app:${BUILD_NUMBER}

          docker push \
            548889528327.dkr.ecr.us-west-2.amazonaws.com/rails-app:${BUILD_NUMBER}
        '''
      }
    }
  }
}
