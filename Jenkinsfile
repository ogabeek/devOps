pipeline {
    agent any

    tools {
       go "1.24.1"
    }

    triggers {
        pollSCM('*/1 * * * *') // Poll Git repository every 1 minute
    }

    stages {
        stage('Unit Test') {
            steps {
                sh "go test -v ./..."
            }
        }
        stage('Build') {
            steps {
                sh "go build main.go"
            }
        }
        stage('Deploy') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'target-ssh-key',
                                                   keyFileVariable: 'ssh_key',
                                                   usernameVariable: 'ssh_user')]) {
                    sh """

chmod +x main

mkdir -p ~/.ssh
ssh-keyscan target >> ~/.ssh/known_hosts

ssh -i ${ssh_key} laborant@target 'sudo systemctl stop main.service'

scp -i ${ssh_key} main ${ssh_user}@target:

scp -i ${ssh_key} main.service ${ssh_user}@target:

ssh -i ${ssh_key} laborant@target 'sudo mv /home/laborant/main.service /etc/systemd/system/main.service'

ssh -i ${ssh_key} laborant@target 'sudo mv /home/laborant/main /opt/main'

ssh -i ${ssh_key} laborant@target 'sudo systemctl daemon-reload'

ssh -i ${ssh_key} laborant@target 'sudo systemctl enable --now main.service'
"""

}
            }
        }
    }
}