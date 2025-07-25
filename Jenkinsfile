pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "yourdockerhubusername/ci-cd-demo"
        EC2_IP = ""  // Will be updated with Terraform output
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/yourusername/devops-ci-cd-demo.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:latest .'
            }
        }
        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub-credentials']) {
                    sh 'docker push $DOCKER_IMAGE:latest'
                }
            }
        }
        stage('Terraform Apply') {
            steps {
                dir('terraform') {
                    sh '''
                      terraform init
                      terraform apply -auto-approve
                      '''
                    script {
                        EC2_IP = sh(script: "terraform output -raw ec2_public_ip", returnStdout: true).trim()
                        echo "EC2 Public IP: ${EC2_IP}"
                    }
                }
            }
        }
        stage('Deploy on AWS EC2') {
            steps {
                sh '''
                  ssh -o StrictHostKeyChecking=no ec2-user@${EC2_IP} \
                  "docker pull $DOCKER_IMAGE:latest && docker stop app || true && docker rm app || true && docker run -d --name app -p 80:3000 $DOCKER_IMAGE:latest"
                '''
            }
        }
    }
}
