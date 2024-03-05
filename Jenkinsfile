pipeline {
    agent any

    environment {
        // Define environment variables
        SERVER_IP = '34.72.227.15'
        REMOTE_DIR = '/home/akarna/sfds'
        LOCAL_DIR = '/var/jenkins_home/sfds' // Specify the local directory where the repository is
        CREDENTIALS_ID = 'jenkins-ssh-credential-id'
    }

    stages {
        stage('Check Connectivity') {
            steps {
                script {
                    try {
                        // Check if the server is reachable via SSH
                        sh "ssh -o BatchMode=yes -o ConnectTimeout=10 akarna@${SERVER_IP} echo 'SSH connection successful'"
                    } catch (Exception e) {
                        // Handle errors during connectivity check
                        echo "Error during SSH connectivity check: ${e.getMessage()}"
                        error("Stopping the build due to SSH connectivity failure")
                    }
                }
            }
        }

        stage('Transfer Files') {
            steps {
                sshagent(credentials: ['jenkins-ssh-credential-id']) {
                    script {
                        try {
                            // Transfer the files from Jenkins server to the remote server
                            sh """
                        echo "Hello"
                        cd /var/jenkins_home/sfds
                        git pull
                        tar --exclude='.git' -czf sfds.tar.gz -C /var/jenkins_home/sfds .
                        scp sfds.tar.gz akarna@${SERVER_IP}:/home/akarna/sfds/
                        ssh akarna@${SERVER_IP} '
                            tar -xzf /home/akarna/sfds/sfds.tar.gz -C /home/akarna/sfds
                            rm /home/akarna/sfds/sfds.tar.gz
                        '
                            """
                        } catch (Exception e) {
                            // Handle errors during file transfer
                            echo "Error during file transfer: ${e.getMessage()}"
                            error("Stopping the build due to file transfer failure")
                        }
                    }
                }
            }
        }

        stage('Docker Operations') {
            steps {
                sshagent(credentials: ['jenkins-ssh-credential-id']) {
                    script {
                        try {
                            // Build and run Docker compose
                            sh """
                                ssh akarna@${SERVER_IP} '
                                    set -e
                                    cd ${REMOTE_DIR}
                                    docker compose up --build -d
                                '
                            """
                        } catch (Exception e) {
                            // Handle errors during Docker compose
                            echo "Error during Docker operations: ${e.getMessage()}"
                            error("Stopping the build due to Docker compose failure")
                        }
                    }
                }
            }
        }

        stage('Verification') {
            steps {
                script {
                    // Verify the deployment
                    sh """
                        ssh akarna@${SERVER_IP} '
                            docker ps
                        '
                    """
                }
            }
        }
    }

    post {
        always {
            // Perform clean-up actions or notifications
            echo 'Pipeline execution is complete.'
        }
        success {
            // Actions to perform on success
            echo 'Deployment successful!'
        }
        failure {
            // Actions to perform on failure
            echo 'Deployment failed. Check logs for details.'
        }
    }
}
