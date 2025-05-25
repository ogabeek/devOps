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
        echo "→ Deploying ${APP_NAME} & ${SERVICE_NAME} to ${TARGET_HOST}"
        sshagent (credentials: ['target-ssh-key']) {
          sh """
            set -e
            echo "→ Uploading files…"
            scp -o StrictHostKeyChecking=no ${APP_NAME} \
                ${SERVICE_NAME} \
                ${DEPLOY_USER}@${TARGET_HOST}:/home/${DEPLOY_USER}/

            echo "→ Running remote service update…"
            ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${TARGET_HOST} << 'EOF'
              sudo systemctl stop ${SERVICE_NAME} || true
              sudo mkdir -p ${REMOTE_APP_DIR}
              sudo mv /home/${DEPLOY_USER}/${APP_NAME}      ${REMOTE_APP_DIR}/${APP_NAME}
              sudo mv /home/${DEPLOY_USER}/${SERVICE_NAME}  /etc/systemd/system/${SERVICE_NAME}
              sudo systemctl daemon-reload
              sudo systemctl enable --now ${SERVICE_NAME}
            EOF
          """
        }
      }
    }
  }
}
