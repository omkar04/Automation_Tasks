pipeline {
    agent any

    parameters {
        string(name: 'SERVER_NAME', defaultValue: '', description: 'Enter the server name or IP address')
        file(name: 'PEM_FILE', description: 'Upload the .pem file')
        text(name: 'PUBLIC_KEY', defaultValue: '', description: 'Enter the public key to be added to authorized_keys')
    }

    environment {
        PEM_FILE_PATH = "${params.PEM_FILE}"
    }

    stages {
        stage('Prepare SSH Key') {
            steps {
                script {
                    // Save the uploaded .pem file to a known location
                    def pemFilePath = "${env.WORKSPACE}\\${params.PEM_FILE}"
                    writeFile file: pemFilePath, text: readFile(params.PEM_FILE)
                }
            }
        }

        stage('Add Public Key to Server') {
            steps {
                script {
                    // Use PowerShell to add the public key to the remote server
                    def pemFilePath = "${env.WORKSPACE}\\${params.PEM_FILE}"
                    def publicKey = params.PUBLIC_KEY
                    def serverName = params.SERVER_NAME

                    bat """
                        powershell.exe -Command \"
                        \$pemFilePath = '${pemFilePath}';
                        \$publicKey = '${publicKey}';
                        \$serverName = '${serverName}';
                        \$sshCommand = \\"echo \$publicKey >> ~/.ssh/authorized_keys\\";
                        ssh -i \$pemFilePath ec2-user@\$serverName \$sshCommand
                        \"
                    """
                }
            }
        }
    }

    post {
        cleanup {
            // Clean up the private key file
            script {
                def pemFilePath = "${env.WORKSPACE}\\${params.PEM_FILE}"
                bat "del ${pemFilePath}"
            }
        }
    }
}
