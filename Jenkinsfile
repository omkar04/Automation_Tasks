pipeline {
    agent any

    parameters {
        string(name: 'SERVER_NAME', defaultValue: 'ubuntu@ec2-52-66-19-141.ap-south-1.compute.amazonaws.com', description: 'Enter the server name or IP address')
        file(name: 'PEM_FILE', description: 'Upload the .pem file')
        text(name: 'PUBLIC_KEY', defaultValue: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC774f7iXx+Ke/IznMARrH4cj395DOFz/Du+5vNlHJIm6qT2zdfZ46VDT7yBTPHS/myDRNFf2dFPEcNdYPUULT47F8VpTL6ixbg8aeNoKz7t33V3dQYXUgeArex6qjDHTMqo59S33cicW7nBMc7V0htUQEXXozTDX82mvqkwQF0CW93gKHWOJsGgtsanw2+JQFdONTcTQcW9CidEB0pRrdOprTAI0OMJzWQOk8YlTiHTzfQEYQegNRxJ0civQCiYVmKon+pB5nd+hxCNESz3mdT2YNqY6cEmBMA+x/P0/UzOauVb8VmKldM4pNVKuNLhoIVdbY+wttxoV3Ew4GoOk34WCmBn5v41sp3kx2maOjqsHc6jb7ewpEl/f+u3huSuRJIg5Me/obqLCi77NLMKE/3dnq2ZNQ1Sp+kXBUITZq4A9DVBBzzIC0LPKf7YxWCrmE55Wt0kjVUVvfsWqgvXl9YKNtrmoaE1dWgA2kPUavfBLoNGyMjG0cG0bT2Pr520tM= shind@Onkars-04', description: 'Enter the public key to be added to authorized_keys')
    }

    environment {
        PEM_FILE_PATH = "${env.WORKSPACE}\\private_key.pem"
    }

    stages {
        stage('Prepare SSH Key') {
            steps {
                script {
                    // Ensure the PEM file path is correctly set
                    def pemFilePath = env.PEM_FILE_PATH
                    writeFile file: pemFilePath, text: readFile(params.PEM_FILE)
                    // Set permissions for the .pem file (use icacls for Windows)
                    bat "icacls ${pemFilePath} /grant:r ${System.getenv('USERNAME')}:R"
                }
            }
        }

        stage('Add Public Key to Server') {
            steps {
                script {
                    // Use PowerShell to add the public key to the remote server
                    def serverName = params.SERVER_NAME
                    def publicKey = params.PUBLIC_KEY.replace("\n", " ").replace("\r", "")
                    def pemFilePath = env.PEM_FILE_PATH

                    bat """
                        powershell.exe -Command \"
                        \$pemFilePath = '${pemFilePath}';
                        \$publicKey = '${publicKey}';
                        \$serverName = '${serverName}';
                        \$sshCommand = 'echo ${publicKey} >> ~/.ssh/authorized_keys';
                        ssh -o StrictHostKeyChecking=no -i \$pemFilePath ec2-user@\$serverName \$sshCommand
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
                def pemFilePath = env.PEM_FILE_PATH
                bat "del /f /q ${pemFilePath}"
            }
        }
    }
}
