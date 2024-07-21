pipeline {
    agent any

   parameters {
        string(name: 'SERVER_NAME', defaultValue: '', description: 'Enter the server name or IP address')
        file(name: 'PEM_FILE', description: 'Upload the .pem file')
        text(name: 'PUBLIC_KEY', defaultValue: '', description: 'Enter the public key to be added to authorized_keys')
    }
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
