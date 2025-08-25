pipeline {
    agent any
    environment {
        EC2_HOST = '3.109.137.50'
        EC2_USER = 'ubuntu'
        SSH_KEY = credentials('ec2-ssh-key')
    }
    stages {
       stage('Build') {
            agent {
                docker {
                    image 'node:22-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    node --version
                    npm --version
                    npm install
                    npm run build
                '''
            }
        }
        stage('Deploy to EC2') {
            steps {
                script {
                    // Transfer files to EC2
                    sh "rsync -avz -e 'ssh -i ${SSH_KEY}' dist/ ${EC2_USER}@${EC2_HOST}:/var/www/html/"
                    
                    // SSH into EC2 and restart services
                    sh "ssh -i ${SSH_KEY} ${EC2_USER}@${EC2_HOST} 'cd /home/ubuntu/backend && git pull && npm install && pm2 restart all'"
                }
            }
        }
    }
}