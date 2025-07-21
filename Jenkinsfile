pipeline {
    agent any

    environment {
        EC2_USER = 'ubuntu'                           // or ec2-user for Amazon Linux
        EC2_HOST = 'ec2-xxx-xxx-xxx-xxx.compute.amazonaws.com'
        PEM_FILE = '/var/lib/jenkins/keys/my-key.pem'  // Jenkins path to your PEM key
        REMOTE_DIR = '/var/www/html'
    }

    stages {
        stage('Prepare') {
            steps {
                echo 'Preparing deployment to EC2...'
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo 'Copying files to EC2...'
                sh '''
                    chmod 400 ${PEM_FILE}
                    scp -i ${PEM_FILE} -o StrictHostKeyChecking=no -r * ${EC2_USER}@${EC2_HOST}:${REMOTE_DIR}
                '''
            }
        }

        stage('Restart Web Server') {
            steps {
                echo 'Restarting Apache/Nginx on EC2...'
                sh '''
                    ssh -i ${PEM_FILE} -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                    sudo systemctl restart apache2 || sudo systemctl restart nginx
                    '
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment to EC2 successful!'
        }
        failure {
            echo 'Deployment to EC2 failed.'
        }
    }
}
