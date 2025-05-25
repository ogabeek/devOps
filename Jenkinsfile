pipeline {
  agent any

  environment {
    TARGET_HOST    = 'target'             // or '172.16.0.3'
    APP_NAME       = 'main'               // your Go binary
    SERVICE_NAME   = 'main.service'
    REMOTE_APP_DIR = '/opt/main'
  }

  tools {
    go '1.24.1'
  }

  triggers {
    pollSCM('H/1 * * * *')
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
    echo "→ [Deploy] Workspace:"
    sh 'ls -R . || true'

    // bind both SSH key _and_ username into env vars
    withCredentials([sshUserPrivateKey(
        credentialsId: 'target-ssh-key',
        keyFileVariable: 'SSH_KEY',
        usernameVariable: 'SSH_USER'
    )]) {
      // turn on bash debugging so you see every single command
      sh '''
        #!/usr/bin/env bash
        set -xe

        echo "→ [Deploy] Deploying ${APP_NAME} & ${SERVICE_NAME} to ${SSH_USER}@${TARGET_HOST}"

        # ensure our binary is executable
        chmod +x ${APP_NAME}

        # stop old service (ignore failures)
        ssh -o StrictHostKeyChecking=no -i "${SSH_KEY}" "${SSH_USER}@${TARGET_HOST}" \
            'sudo systemctl stop ${SERVICE_NAME} || true'

        # copy both the binary and the unit file
        scp -o StrictHostKeyChecking=no -i "${SSH_KEY}" \
            "${APP_NAME}" \
            "${SERVICE_NAME}" \
            "${SSH_USER}@${TARGET_HOST}:/home/${SSH_USER}/"

        # one SSH session to move files & restart
        ssh -o StrictHostKeyChecking=no -i "${SSH_KEY}" "${SSH_USER}@${TARGET_HOST}" << 'EOF'
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
    }
  }
}
