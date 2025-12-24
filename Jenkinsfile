pipeline {
  agent any

  environment {
    // --- Git ---
    GIT_URL      = "https://github.com/kranthi31/dockejava.git"
    GIT_BRANCH   = "main"
    GIT_CREDS_ID = ""   // set Jenkins credentialsId if private repo, else keep ""

    // --- AWS/ECR ---
    AWS_REGION   = "us-west-2"
    AWS_ACCOUNT  = "548889528327"
    ECR_REPO     = "rails-app"
    ECR_REGISTRY = "${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    IMAGE_TAG    = "${BUILD_NUMBER}"
  }

  options {
    skipDefaultCheckout(true)   // prevent Jenkins from doing an extra checkout automatically
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
            userRemoteConfigs: [remoteCfg],
            extensions: [
              [$class: 'CleanBeforeCheckout']   // clean workspace before checkout
            ]
          ])
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          if (isUnix()) {
            sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
          } else {
            bat "docker build -t %ECR_REPO%:%IMAGE_TAG% ."
          }
        }
      }
    }

    stage('Login to ECR') {
      steps {
        script {
          if (isUnix()) {
            sh """
              aws ecr get-login-password --region ${AWS_REGION} | \
              docker login --username kranthi-test --password-stdin ${ECR_REGISTRY}
            """
          } else {
            // Windows: no backslash line continuation, use cmd piping
            bat """
              aws ecr get-login-password --region %AWS_REGION% | docker login --username AWS --password-stdin %ECR_REGISTRY%
            """
          }
        }
      }
    }

    stage('Tag & Push') {
      steps {
        script {
          if (isUnix()) {
            sh """
              docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}
              docker push ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}
            """
          } else {
            bat """
              docker tag %ECR_REPO%:%IMAGE_TAG% %ECR_REGISTRY%/%ECR_REPO%:%IMAGE_TAG%
              docker push %ECR_REGISTRY%/%ECR_REPO%:%IMAGE_TAG%
            """
          }
        }
      }
    }
  }
}
