pipeline {
    agent any

    tools {
       go "1.24.1"
    }

    triggers{
        pollSCM('*/1 * * * *') //poll git every 1 min
    }

    stages {
        stage('Build') {
            steps {
                sh "go build project/main.go"
            }
        }
    }
}
