pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID     = credentials('aws-account-id')
        AWS_REGION         = 'us-east-1'
        ECR_REPO_NAME      = 'demo-project'
        ECR_REGISTRY       = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_NAME         = "${ECR_REGISTRY}/${ECR_REPO_NAME}"
        IMAGE_TAG          = "${BUILD_NUMBER}"

        // SonarQube settings
        SONAR_PROJECT_KEY  = 'devsecops-project'
    }

    tools {
        maven 'Maven-3.9'
        jdk   'JDK-17'
    }

    stages {

         stage('Checkout') {
            steps {
                echo '===== Pulling source code from GitHub ====='
                checkout scm
            }
        }
        
        stage('Build Backend') {
            steps {
                sh 'docker build -t -t ${IMAGE_NAME}:${IMAGE_TAG}-backend ./backend'
            }
        }
        stage('Build Frontend') {
            steps {
                sh 'docker build -t -t ${IMAGE_NAME}:${IMAGE_TAG}-frontend ./frontend'
            }
        }
       stage('Push Images') {
            steps {
                sh 'docker push ${IMAGE_NAME}:${IMAGE_TAG}-backend'
                sh 'docker push ${IMAGE_NAME}:${IMAGE_TAG}-frontend'
            }
        }
    }
}
