pipeline {
  agent any

  // Move all of your constants into one place
  environment {
    TARGET_HOST   = 'target'                    // or '172.16.0.3'
    APP_NAME      = 'main'                      // your Go binary
    SERVICE_NAME  = 'main.service'              // the Systemd unit file
    REMOTE_APP_DIR= '/opt/main'                 // where your binary lives
  }

  tools {
    go '1.24.1'
  }

  triggers {
    // Poll your Git repo every minute
    pollSCM('H/1 * * * *')
  }

  stages {
    stage('Unit Test') {
      steps {
        sh 'go test -v ./...'
      }
    }

    stage('Build') {
      steps {
        // Build your binary with the name in APP_NAME
        sh "go build -o ${APP_NAME} main.go"
      }
    }

    stage('Deploy') {
      steps {
        // Bind your SSH key and user from Jenkins Credentials
        withCredentials([sshUserPrivateKey(
            credentialsId: 'target-ssh-key',
            keyFileVariable: 'SSH_KEY_PATH',
            usernameVariable: 'DEPLOY_USER'
        )]) {
          sh """
            #!/bin/bash
            set -e

            # Make sure our binary is executable
            chmod +x ${APP_NAME}

            # Arrange known_hosts so SSH won't prompt
            mkdir -p ~/.ssh
            ssh-keyscan -H ${TARGET_HOST} >> ~/.ssh/known_hosts

            # Copy both the binary and the systemd unit file
            scp -i ${SSH_KEY_PATH} ${APP_NAME}      ${DEPLOY_USER}@${TARGET_HOST}:/home/${DEPLOY_USER}/
            scp -i ${SSH_KEY_PATH} ${SERVICE_NAME}  ${DEPLOY_USER}@${TARGET_HOST}:/home/${DEPLOY_USER}/

            # One SSH session to do all the remote admin work
            ssh -i ${SSH_KEY_PATH} ${DEPLOY_USER}@${TARGET_HOST} << 'EOF'
              set -e

              # Stop the old service (ignore if not running)
              sudo systemctl stop ${SERVICE_NAME} || true

              # Make sure remote dirs exist
              sudo mkdir -p ${REMOTE_APP_DIR}

              # Move files into place
              sudo mv /home/${DEPLOY_USER}/${APP_NAME}      ${REMOTE_APP_DIR}/${APP_NAME}
              sudo mv /home/${DEPLOY_USER}/${SERVICE_NAME}  /etc/systemd/system/${SERVICE_NAME}

              # Reload systemd and bring up the new service
              sudo systemctl daemon-reload
              sudo systemctl enable --now ${SERVICE_NAME}
            EOF
          """
        }
      }
    }
  }
}
