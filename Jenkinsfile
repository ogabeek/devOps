pipeline {
  agent any

  environment {
    TARGET_HOST    = 'target'             // or '172.16.0.3'
    APP_NAME       = 'main'               // your Go binary
    SERVICE_NAME   = 'main.service'       // the Systemd unit file
    REMOTE_APP_DIR = '/opt/main'          // where your binary lives on the target
  }

  tools {
    go '1.24.1'
  }

  triggers {
    pollSCM('H/1 * * * *')                // Poll Git repo every minute
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Unit Test') {
      steps {
        sh 'go test -v ./...'
      }
    }

    stage('Build') {
      steps {
        sh "go build -o ${APP_NAME} main.go"
      }
    }

    stage('Deploy') {
      steps {
        echo "→ [Deploy] Starting at ${new Date().format("HH:mm:ss")}"
        echo "→ [Deploy] Workspace contents:"
        sh 'ls -R . || true'

        withCredentials([sshUserPrivateKey(
            credentialsId: 'target-ssh-key',
            keyFileVariable: 'SSH_KEY',
            usernameVariable: 'SSH_USER'
        )]) {
          sh '''
            #!/usr/bin/env bash
            set -xe

            echo "→ [Deploy] Deploying ${APP_NAME} & ${SERVICE_NAME} to ${SSH_USER}@${TARGET_HOST}"

            # Ensure binary is executable
            chmod +x ${APP_NAME}

            # Stop old service (ignore error if not running)
            ssh -o StrictHostKeyChecking=no -i "${SSH_KEY}" \
                "${SSH_USER}@${TARGET_HOST}" \
                'sudo systemctl stop ${SERVICE_NAME} || true'

            # Copy both the binary and the service unit file
            scp -o StrictHostKeyChecking=no -i "${SSH_KEY}" \
                "${APP_NAME}" \
                "${SERVICE_NAME}" \
                "${SSH_USER}@${TARGET_HOST}:/home/${SSH_USER}/"

            # Single SSH session to move files & restart
            ssh -o StrictHostKeyChecking=no -i "${SSH_KEY}" \
                "${SSH_USER}@${TARGET_HOST}" << 'EOF'
              set -xe
              sudo mkdir -p ${REMOTE_APP_DIR}
              sudo mv /home/${SSH_USER}/${APP_NAME}      ${REMOTE_APP_DIR}/${APP_NAME}
              sudo mv /home/${SSH_USER}/${SERVICE_NAME}  /etc/systemd/system/${SERVICE_NAME}
              sudo systemctl daemon-reload
              sudo systemctl enable --now ${SERVICE_NAME}
EOF
          '''
        }
      }
    }
  }
}
