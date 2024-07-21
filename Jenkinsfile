pipeline {
    agent any
    
    environment {
        PEM_FILE_PATH = "${env.WORKSPACE}/${params.PEM_FILE}"
    }

    stages {
        stage('Prepare SSH Key') {
            steps {
                script {
                    // Save the uploaded .pem file to a known location
                    writeFile file: 'private_key.pem', text: readFile(params.PEM_FILE)
                    sh 'chmod 400 private_key.pem'
                }
            }
        }

        stage('Add Public Key to Server') {
            steps {
                script {
                    // Use the ssh-agent plugin to handle SSH credentials
                    sshagent (credentials: ['your-ssh-credential-id']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no -i private_key.pem ec2-user@${params.SERVER_NAME} 'echo "${params.PUBLIC_KEY}" >> ~/.ssh/authorized_keys'
                        """
                    }
                }
            }
        }
    }

    post {
        cleanup {
            // Clean up the private key file
            sh 'rm -f private_key.pem'
        }
    }
}
