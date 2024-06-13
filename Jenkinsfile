pipeline {
    agent any

    environment {
        registry = "atchyuth417/myapp"
        registryCredential = 'dockerhub-credentials'
        dockerImage = ''
        kubeconfigId = 'kubeconfig'
    }

    stages {
        stage('Cloning Git') {
            steps {
                script {
                    echo "Cloning the Git repository..."
                    git 'https://github.com/atchyuth417/project.git'
                }
            }
        }
        
        stage('Building Docker Image') {
            steps {
                script {
                    echo "Building the Docker image..."
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        
        stage('Running Unit Tests') {
            steps {
                script {
                    echo "Running unit tests..."
                    dockerImage.inside {
                        sh 'npm install'
                        sh 'npm test'
                    }
                }
            }
        }
        
        stage('Push to DockerHub') {
            steps {
                script {
                    echo "Pushing the Docker image to DockerHub..."
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Deploying to Kubernetes..."
                    kubernetesDeploy configs: 'k8s-deployment.yaml', kubeconfigId: kubeconfigId
                }
            }
        }
    }
    
    post {
        always {
            script {
                echo "Cleaning up the workspace..."
                cleanWs()
            }
        }
    }
}
