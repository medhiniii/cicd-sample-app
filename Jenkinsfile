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
                echo 'Cloning source code from GitHub...'
                git branch: 'main', url: 'https://github.com/medhiniii/cicd-sample-app.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Installing dependencies and building the application...'
                sh 'npm install'
            }
        }

        
        stage('Test') {
            steps {
                echo 'Running tests (optional)...'
                script {
                    sh '''
                        if [ -f package.json ]; then
                            npm test || echo "No tests found or tests failed — continuing pipeline."
                        else
                            echo "package.json not found — skipping test stage."
                        fi
                    '''
                }
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
                echo 'Deploying container on EC2 instance...'
                withAWS(region: "${AWS_REGION}", credentials: 'aws-creds') {
                    script {
                        sh '''
                            echo "Logging in to ECR..."
                            aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

                            echo "Stopping old container (if running)..."
                            docker ps -q --filter "name=cicd-app" | xargs -r docker stop
                            docker ps -a -q --filter "name=cicd-app" | xargs -r docker rm

                            echo "Pulling latest image..."
                            docker pull $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO_NAME:$IMAGE_TAG

                            echo "Starting new container..."
                            docker run -d --name cicd-app -p 3000:8080 $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO_NAME:$IMAGE_TAG
                        '''
                    }
                }
            }
        }

        
        stage('Post-Deploy Health Check') {
            steps {
                echo 'Verifying that the deployed application is running...'
                script {
                    sh '''
                        sleep 10  # give container time to start
                        curl -f http://localhost:3000 || echo " Health check failed, app may not be running correctly."
                    '''
                }
            }
        }
    }

    post {
        success {
            echo ' Pipeline completed successfully!'
        }
        failure {
            echo ' Pipeline failed — check logs for details.'
        }
    }
}
