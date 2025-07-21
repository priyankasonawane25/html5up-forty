pipeline {
    agent any

    environment {
        EC2_HOST = 'ec2-13-203-159-118.ap-south-1.compute.amazonaws.com'  // Your EC2 public DNS
        EC2_USER = 'ubuntu'                                               // Default user for Ubuntu EC2
        REMOTE_DIR = '/var/www/html'                                      // Web server root on EC2
        SSH_CREDENTIAL_ID = 'ec2-ssh-key'                                 // ID of SSH key in Jenkins Credentials
    }

    stages {
        stage('Prepare') {
            steps {
                echo 'Preparing to deploy static website to EC2...'
            }
        }

        stage('Deploy Files to EC2') {
            steps {
                echo "Using SSH credentials ID: ${SSH_CREDENTIAL_ID}"
                sshagent(credentials: ["${SSH_CREDENTIAL_ID}"]) {
                    sh '''
                        echo "Copying website files to EC2..."
                        scp -o StrictHostKeyChecking=no -r * ${EC2_USER}@${EC2_HOST}:${REMOTE_DIR}
                    '''
                }
            }
        }

        stage('Restart Web Server on EC2') {
            steps {
                sshagent(credentials: ["${SSH_CREDENTIAL_ID}"]) {
                    sh '''
                        echo "Restarting web server on EC2..."
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            if systemctl is-active --quiet apache2; then
                                sudo systemctl restart apache2
                            elif systemctl is-active --quiet nginx; then
                                sudo systemctl restart nginx
                            else
                                echo "No web server found to restart."
                            fi
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
            echo '❌ Deployment to EC2 failed. Check logs for details.'
        }
    }
}
