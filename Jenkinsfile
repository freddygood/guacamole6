/*
Jenkinsfile for multi-branch task for mp30-ui
*/

pipeline {
    agent {
        label 'master'
    }

    environment {
        MAPPING_FILE_NAME = 'mapping/mp30/ui-mapping.yml'
        TASKS_TO_EXCLUDE = 'mp30-ui-deploy'

        PROFILES_DIR = 'profiles'
        PROFILES_GIT_REPO='git@github.com:media20/profiles.git'
        PROFILES_GIT_BRANCH = "master"
        PROFILES_GIT_CREDENTIALS_ID='d335f9d6-8f0a-48b6-8415-6f0384db0121'
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
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: PROFILES_GIT_BRANCH]],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [
                        [$class: 'PathRestriction', includedRegions: MAPPING_FILE_NAME],
                        [$class: 'RelativeTargetDirectory', relativeTargetDir: PROFILES_DIR],
                        [$class: 'CloneOption', depth: 0, noTags: false, reference: '', shallow: true]
                    ],
                    submoduleCfg: [],
                    userRemoteConfigs: [[
                        credentialsId: PROFILES_GIT_CREDENTIALS_ID,
                        url: PROFILES_GIT_REPO
                    ]]
                ])

                script {
                    def mapping = []
                    if (fileExists( "${env.PROFILES_DIR}/${env.MAPPING_FILE_NAME}" )) {
                        mapping = readYaml file: "${env.PROFILES_DIR}/${env.MAPPING_FILE_NAME}"
                    }
                    println "Got task by branch mapping => ${mapping}"
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
                        echo "Processing task '${task}'"
                        if (task in env.TASKS_TO_EXCLUDE.split(',')) {
                            echo "Task '${task}' is excluded. Skipping.."
                        } else {
                            echo "Task '${task}' is scheduled on '${env.BRANCH_NAME}'."
                            builds[task] = {
                                echo "Starting the task '${task}' on branch '${env.BRANCH_NAME}'"
                                build job: task, parameters: [ string(name: 'BRANCH', value: env.BRANCH_NAME) ]
                            }
                        }
                    }
                    println "Got run build tasks => ${builds.keySet()}"
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
