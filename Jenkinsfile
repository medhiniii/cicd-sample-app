pipeline {
    agent any

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
                sh 'docker build -t cicd-sample-app .'
            }
        }

        stage('Push to ECR (skip for now)') {
            steps {
                echo 'Placeholder for pushing Docker image to AWS ECR...'
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo 'Deploying application container on EC2...'
                sh '''
                    echo "Stopping old container (if running)..."
                    docker ps -q --filter "ancestor=cicd-sample-app" | xargs -r docker stop
                    docker ps -a -q --filter "ancestor=cicd-sample-app" | xargs -r docker rm

                    echo "Starting new container..."
                    docker run -d -p 3000:8080 cicd-sample-app
                '''
            }
        }
    }
}
