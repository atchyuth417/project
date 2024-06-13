pipeline {
    agent any

    environment {
        registry = "atchyuth417/myapp"   // Update to your DockerHub username and repository
        registryCredential = 'dockerhub-credentials'             // Ensure this matches the ID of your DockerHub credentials in Jenkins
        dockerImage = ''
        kubeconfigCredentialId = 'kubeconfig'                  // Ensure this matches the ID of your Kubernetes kubeconfig in Jenkins
    }

    stages {
        stage('Cloning Git') {
            steps {
                script {
                    echo "Cloning the Git repository..."
                    git branch: 'main',
                        url: 'https://github.com/atchyuth417/project.git'  // Update to your actual repository URL
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
                        sh 'npm test'   // Update if your test command differs
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
                    
                    // Set kubectl context using kubeconfig from Jenkins credentials
                    withCredentials([file(credentialsId: kubeconfigCredentialId, variable: 'KUBECONFIG_FILE')]) {
                        sh "kubectl --kubeconfig=${KUBECONFIG_FILE} config use-context kubernetes-admin@Maigha"
                        sh "kubectl --kubeconfig=${KUBECONFIG_FILE} apply -f k8s-deployment.yaml"
                        sh "kubectl --kubeconfig=${KUBECONFIG_FILE} rollout status deployment/myapp-deployment"
                    }
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
