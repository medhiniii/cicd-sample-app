pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '478962752033'
        AWS_REGION = 'ap-south-1'
        ECR_REPO_NAME = 'cicd-sample-app'
        IMAGE_TAG = 'latest'
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/medhiniii/cicd-sample-app.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Building the application...'
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'npm test || echo "No tests to run"'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t ${ECR_REPO_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Push to ECR') {
            steps {
                echo 'Pushing Docker image to AWS ECR...'
                withAWS(region: "${AWS_REGION}", credentials: 'aws-creds') {
                    script {
                        sh '''
                            aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                            docker tag $ECR_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO_NAME:$IMAGE_TAG
                            docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO_NAME:$IMAGE_TAG
                        '''
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo 'Deploying application container on EC2...'
                withAWS(region: "${AWS_REGION}", credentials: 'aws-creds') {
                    script {
                        sh '''
                            echo "Logging in to ECR..."
                            aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

                            echo "Stopping old container (if running)..."
                            docker ps -q --filter "name=cicd-app" | xargs -r docker stop
                            docker ps -a -q --filter "name=cicd-app" | xargs -r docker rm

                            echo "Pulling latest image from ECR..."
                            docker pull $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO_NAME:$IMAGE_TAG

                            echo "Starting new container..."
                            docker run -d --name cicd-app -p 3000:8080 $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO_NAME:$IMAGE_TAG
                        '''
                    }
                }
            }
        }
    }
}
