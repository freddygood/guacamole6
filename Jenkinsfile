/*
Jenkinsfile for multi-branch task for mp30-ui
*/

pipeline {
    agent {
        label 'master'
    }

    environment {
        TASK_BY_BRANCH_MAP_FILE_NAME = './taskByBranchMap.yml'
    }

    options {
        disableConcurrentBuilds()
        buildDiscarder(
            logRotator(numToKeepStr: '30')
        )
    }

    stages {
        stage('Prepare') {
            steps {
                script {
                    def mapping = []
                    if (fileExists(TASK_BY_BRANCH_MAP_FILE_NAME)) {
                        mapping = readYaml file: TASK_BY_BRANCH_MAP_FILE_NAME
                    }
                    println "Got task by branch mapping => ${mapping}"
                    env.TASKS = mapping.branches[env.BRANCH_NAME].join(',')
                }
                echo "Found tasks '${env.TASKS}' for branch '${env.BRANCH_NAME}'"
            }
        }

        stage('Build') {
            steps {
                sh "printenv"
                script {
                    def builds = [:]
                    for (t in env.TASKS.split(',')) {
                        builds[t] = {
                            node {
                                echo "Starting the task '${t}' on branch '${env.BRANCH_NAME}'"
                            }
                        }
                    }
                    println "Got run build tasks => ${builds}"

                    parallel(builds)
                }
            }
        }
    }

    post {
        always {
            updateGithubCommitStatus()
        }
    }
}

def getRepoURL() {
    sh "git config --get remote.origin.url > .git/remote-url"
    return readFile(".git/remote-url").trim()
}

def getCommitSha() {
    sh "git rev-parse HEAD > .git/current-commit"
    return readFile(".git/current-commit").trim()
}

def updateGithubCommitStatus(build = currentBuild) {
    // workaround https://issues.jenkins-ci.org/browse/JENKINS-38674
    repoUrl = getRepoURL()
    commitSha = getCommitSha()

    step([
        $class: 'GitHubCommitStatusSetter',
        reposSource: [$class: "ManuallyEnteredRepositorySource", url: repoUrl],
        commitShaSource: [$class: "ManuallyEnteredShaSource", sha: commitSha],
        errorHandlers: [[$class: 'ShallowAnyErrorHandler']],
        statusResultSource: [
            $class: 'ConditionalStatusResultSource',
            results: [
                [$class: 'BetterThanOrEqualBuildResult', result: 'SUCCESS', state: 'SUCCESS', message: build.description],
                [$class: 'BetterThanOrEqualBuildResult', result: 'FAILURE', state: 'FAILURE', message: build.description],
                [$class: 'AnyBuildResult', state: 'FAILURE', message: 'Loophole']
            ]
        ]
    ])
}
