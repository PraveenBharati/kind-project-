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
         stage('SonarQube Analysis') {
    steps {
        echo '===== Running SonarQube code quality scan ====='
        withSonarQubeEnv('SonarQube') {
            sh '''
            sonar-scanner \
              -Dsonar.projectKey=devsecops-project \
              -Dsonar.projectName="devsecops-project" \
              -Dsonar.sources=. \
              -Dsonar.exclusions=**/node_modules/**,**/*.test.js \
              -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
            '''
        }
    }
}

        stage('Quality Gate') {
    steps {
        echo '===== Waiting for SonarQube quality gate result ====='
        timeout(time: 5, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
    }
}
        
        stage('Build Backend') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG}-backend ./backend'
            }
        }
        stage('Build Frontend') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG}-frontend ./frontend'
            }
        }
       stage('Push Images') {
           steps {
                echo '===== Pushing image to AWS ECR ====='
                withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
                    sh '''
                        aws ecr get-login-password --region ${AWS_REGION} | \
                          docker login --username AWS --password-stdin ${ECR_REGISTRY}

                        docker push ${IMAGE_NAME}:${IMAGE_TAG}-backend
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}-frontend
                    '''
                }
            }
        }
    }
}
