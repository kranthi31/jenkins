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

    // Jenkins AWS credentialsId (Kind: AWS Credentials)
    AWS_CREDS_ID = "aws-ecr-creds"
  }

  options {
    skipDefaultCheckout(true)
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
            extensions: [[$class: 'CleanBeforeCheckout']]
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
