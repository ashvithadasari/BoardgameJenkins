pipeline {
    agent any

    environment {
        IMAGE_NAME = "board-app"
        DOCKERHUB_USER = "YOUR_DOCKERHUB_USERNAME"
        EKS_CLUSTER = "YOUR_EKS_CLUSTER"
        AWS_REGION = "ap-south-1"
    }

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/YOUR_GITHUB_REPO.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Scan') {
            steps {
                sh '''
                mvn sonar:sonar \
                -Dsonar.projectKey=Boardgame \
                -Dsonar.projectName=Boardgame \
                -Dsonar.host.url=http://localhost:9000 \
                -Dsonar.login=YOUR_SONAR_TOKEN
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy fs .'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $DOCKERHUB_USER/$IMAGE_NAME:latest .'
            }
        }

        stage('Docker Hub Login') {
            steps {
                sh '''
                echo "YOUR_DOCKERHUB_PASSWORD" | docker login \
                -u $DOCKERHUB_USER --password-stdin
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh '''
                docker push $DOCKERHUB_USER/$IMAGE_NAME:latest
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                aws eks update-kubeconfig \
                --region $AWS_REGION \
                --name $EKS_CLUSTER

                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}