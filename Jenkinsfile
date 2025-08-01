pipeline {
    agent any

    environment {
        EC2_HOST = '13.204.77.154'
        SSH_CREDENTIALS_ID = 'pem_key'
        REMOTE_USER = 'ubuntu'
        REMOTE_PATH = '/home/ubuntu/app'
        WEB_ROOT = '/var/www/html'
        SERVER = 'nginx'
    }

    stages {
        stage('Build') {
            steps {
                echo "Installing dependencies..."
                sh 'npm install'
                echo "Building the project..."
                sh 'npm run build'
                echo "Build completed successfully."
            }
        }

        stage('Deploy') {
            steps {
                echo "Starting deployment..."
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]) {
                    sh """
                        echo "Creating remote directory"
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${EC2_HOST} "mkdir -p ${REMOTE_PATH}"

                        echo "Copying distribution to remote server"
                        scp -o StrictHostKeyChecking=no -r dist/* ${REMOTE_USER}@${EC2_HOST}:${REMOTE_PATH}/

                        echo "Deploying files to web root and restarting nginx"
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${EC2_HOST} "
                            sudo rm -rf ${WEB_ROOT}/*
                            sudo cp -r ${REMOTE_PATH}/* ${WEB_ROOT}/
                            sudo systemctl restart ${SERVER}
                            echo 'Deployment completed successfully.'
                        "
                    """
                }
            }
        }
    }
}