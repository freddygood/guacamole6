
pipeline {
    agent {
        label 'master'
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
                sh "printenv"
            }
        }

        stage('Select branch') {
            // when {
            //     expression {
            //         return { getStage() }
            //     }
            // }
            steps {
                script {
                    def mapping = []
                    def fileName = "./taskByBranchMap.yml"
                    println "fileName: ${fileName}"
                    if (fileExists(fileName)) {
                      mapping = readYaml file: fileName
                    }
                    println "mapping => ${mapping}"
                    env.TASK = mapping.branches[branch]
                    println "branch => ${env.TASK}"
                }
                echo "Found task '${env.TASK}' for branch '${env.BRANCH_NAME}'"
            }
        }
        // stage('Build') {
        //     parallel {
        //         stage ('production') {
        //             when {
        //                 branch 'master'
        //             }
        //             steps {
        //               echo "build ${BRANCH_NAME} to production"
        //             }
        //         }
        //         stage ('dev') {
        //             when {
        //                 branch 'test1'
        //             }
        //             steps {
        //               echo "build ${BRANCH_NAME} to dev"
        //             }
        //         }
        //         stage ('dev1') {
        //             when {
        //                 branch 'test2'
        //             }
        //             steps {
        //               echo "build ${BRANCH_NAME} to dev2"
        //             }
        //         }
        //     }
        // }
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
