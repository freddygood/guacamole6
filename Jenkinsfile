/*
Jenkinsfile for multi-branch task for mp30-ui
*/

pipeline {
    agent {
        label 'master'
    }

    environment {
        TASK_BY_BRANCH_MAP_FILE_NAME = './taskByBranchMap.yml'
        TASK_TO_EXCLUDE = 'mp30-ui-deploy'
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
                    // env.TASKS = mapping.branches[env.BRANCH_NAME].join(',')
                    env.TASKS = mapping.tasks.findAll { it.value == env.BRANCH_NAME }.keySet().join(',')

                }
                echo "Found tasks '${env.TASKS}' for branch '${env.BRANCH_NAME}'"
            }
        }

        stage('Build') {
            steps {
                script {
                    def builds = [:]
                    for (task in env.TASKS.split(',')) {
                        if (task in TASK_TO_EXCLUDE.split(',')) {
                            echo "Task '${task}' is excluded. Skipping.."
                        } else {
                            builds[task] = {
                                node {
                                    echo "Starting the task '${task}' on branch '${env.BRANCH_NAME}'"
                                }
                            }
                        }
                    }

                    println "Got run build tasks => ${builds.keySet()}"

                    // parallel(builds)
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
