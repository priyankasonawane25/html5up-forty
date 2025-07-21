pipeline {
    agent any

    environment {
        EC2_HOST = 'ec2-13-203-159-118.ap-south-1.compute.amazonaws.com'  // Replace with your EC2 Public DNS
        REMOTE_DIR = '/var/www/html'                                      // Path to deploy on EC2
        EC2_USER = 'ubuntu'                                               // Change to 'ec2-user' if using Amazon Linux
    }

    stages {
        stage('Preparation') {
            steps {
                echo '📦 Preparing deployment to EC2...'
            }
        }

        stage('Deploy Files to EC2') {
            steps {
                echo '🚀 Deploying files to EC2...'
                sshagent (credentials: ['ec2-key']) { // 'ec2-key' is the ID of SSH private key in Jenkins
                    sh '''
                        scp -o StrictHostKeyChecking=no -r * ${EC2_USER}@${EC2_HOST}:${REMOTE_DIR}
                    '''
                }
            }
        }

        stage('Restart Web Server') {
            steps {
                echo '🔁 Restarting web server on EC2...'
                sshagent (credentials: ['ec2-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            sudo systemctl restart apache2 || sudo systemctl restart nginx
                        '
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment to EC2 successful!'
        }
        failure {
            echo '❌ Deployment to EC2 failed.'
        }
    }
}
