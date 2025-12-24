pipeline {
  agent any

  environment {
    // --- Git ---
    GIT_URL      = "https://github.com/kranthi31/dockejava.git"
    GIT_BRANCH   = "main"
    GIT_CREDS_ID = ""   // put Jenkins credentialsId here OR leave blank for public repo

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
        script {
          def remoteCfg = [url: env.GIT_URL]
          if (env.GIT_CREDS_ID?.trim()) {
            remoteCfg.credentialsId = env.GIT_CREDS_ID
          }

          checkout([$class: 'GitSCM',
            branches: [[name: "*/${env.GIT_BRANCH}"]],
            userRemoteConfigs: [remoteCfg]
          ])
        }
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
