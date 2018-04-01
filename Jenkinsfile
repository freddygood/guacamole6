
pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        buildDiscarder(
            logRotator(numToKeepStr: '30')
        )
    }

    stages {
        stage('Build') {
            steps {
                sh "set"
                sh "uname -a"
                sh "echo test1"
            }
        }
    }

    post {
        always {
            sh "set"
        }
    }
}
