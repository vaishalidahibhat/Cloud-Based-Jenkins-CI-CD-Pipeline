pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "your-dockerhub-username/cloud-jenkins-app"
        K8S_DEPLOYMENT = "cloud-jenkins-deployment"
        AWS_S3_BUCKET = "your-s3-bucket-name"
    }

    stages {

        stage('Clone Repository') {
            steps {
                script {
                    echo "Cloning GitHub repository..."
                    git branch: 'main', url: 'https://github.com/vaishalidahibhat/Cloud-Based-Jenkins-CI-CD-Pipeline.git'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'sudo apt-get update && sudo apt-get install -y docker.io kubectl'
            }
        }

        stage('Build & Test Application') {
            steps {
                script {
                    echo "Building application..."
                    sh 'mvn clean package'
                    echo "Running unit tests..."
                    sh 'mvn test'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    sh "docker build -t ${DOCKER_IMAGE}:latest ."
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                script {
                    echo "Pushing Docker image to DockerHub..."
                    withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                        sh "docker push ${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Deploying application to Kubernetes..."
                    sh """
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    """
                }
            }
        }

        stage('Run Functional Tests') {
            steps {
                script {
                    echo "Running post-deployment tests..."
                    sh 'curl -f http://your-k8s-service-url/health || exit 1'
                }
            }
        }

        stage('Upload Logs to S3') {
            steps {
                script {
                    echo "Uploading logs to AWS S3..."
                    sh "aws s3 cp logs/app.log s3://${AWS_S3_BUCKET}/logs/"
                }
            }
        }

    }

    post {
        success {
            echo "Deployment Successful! 🎉"
        }
        failure {
            echo "Deployment Failed! ❌"
            sh "exit 1"
        }
    }
}
