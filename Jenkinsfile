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
                    ls -lah dist || echo "dist folder not found"
                '''
            }
        }
        
        stage('Deploy to EC2') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'SSH_KEY_FILE')]) {
                        sh """
                            chmod 600 ${SSH_KEY_FILE}

                            # Sync into a temporary folder first
                            rsync -avz -e 'ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no' dist/ ${EC2_USER}@${EC2_HOST}:/tmp/dist/

                            # Move files with sudo
                            ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                                sudo rm -rf /var/www/html/*
                                sudo cp -r /tmp/dist/* /var/www/html/
                                sudo chown -R www-data:www-data /var/www/html/
                            '

                            # Restart backend
                            ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                                cd /home/ubuntu/backend && git pull && npm install && pm2 restart all
                            '
                        """
                    }
                }
            }
        }

    }
}
