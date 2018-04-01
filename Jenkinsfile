
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
            }
        }
    }

    post {
        always {
            sh "set"
        }
    }
}
