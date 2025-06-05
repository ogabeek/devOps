pipeline {
    agent any

    tools { go "1.24.1" }   // Make sure Jenkins has a Go tool named “1.24.1”

    triggers {
        pollSCM('H/1 * * * *') // poll Git every minute
    }

    stages {
        stage('Unit Test') {
            steps {
                sh "go test -v ./..."
            }
        }
        stage('Build Binary') {
            steps {
                sh "go build -o main main.go"
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build . --tag ttl.sh/myapp:1h"
            }
        }
        stage('Push Docker Image') {
            steps {
                sh "docker push ttl.sh/myapp:1h"
            }
        }
    }
}
