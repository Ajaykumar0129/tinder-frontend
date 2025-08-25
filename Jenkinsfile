pipeline {
    agent any
    environment {
        EC2_HOST = '3.109.137.50'
        EC2_USER = 'ubuntu'
        SSH_KEY = credentials('ec2-ssh-key')
    }
    stages {    
        stage('Build Frontend') {
            agent {
                docker {
                    image 'node:22-alpine'
                    reuseNode true
                    args '-u root:root'
                }
            }
            steps {
                // Install required tools in the container
                sh 'apk add --no-cache rsync openssh-client'
                sh '''
                        node --version
                        npm install
                        npm run build
                    '''
            }
        }
        
        stage('Deploy to EC2') {
            steps {
                script {
                        sh "rsync -avz -e 'ssh -o StrictHostKeyChecking=no' frontend/dist/ ${EC2_USER}@${EC2_HOST}:/var/www/html/"
                        sh "ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} 'cd /home/ubuntu/backend && git pull && npm install && pm2 restart all'"
                }
            }
        }
    }
}