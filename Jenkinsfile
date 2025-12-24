pipeline {
  agent any

  environment {
    // --- Git in script ---
    GIT_URL      = "https://github.com/ORG/REPO.git"
    GIT_BRANCH   = "main"
    GIT_CREDS_ID = "git-creds-id"   // Jenkins Credentials ID (PAT or SSH key)

    // --- AWS/ECR ---
    AWS_REGION   = "us-west-2"
    AWS_ACCOUNT  = "548889528327"
    ECR_REPO     = "rails-app"
    ECR_REGISTRY = "${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    IMAGE_TAG    = "${BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout([$class: 'GitSCM',
          branches: [[name: "*/${env.GIT_BRANCH}"]],
          userRemoteConfigs: [[
            url: env.GIT_URL,
            credentialsId: env.GIT_CREDS_ID
          ]]
        ])
      }
    }

    stage('Build Docker Image') {
      steps {
        sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
      }
    }

    stage('Login to ECR') {
      steps {
        sh """
          aws ecr get-login-password --region ${AWS_REGION} | \
          docker login --username AWS --password-stdin ${ECR_REGISTRY}
        """
      }
    }

    stage('Tag & Push') {
      steps {
        sh """
          docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}
          docker push ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}
        """
      }
    }
  }
}
