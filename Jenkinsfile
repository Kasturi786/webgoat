pipeline {
    agent any

    options {
        gitLabConnection('gitlab')
    }

    stages {
        stage("build") {
            steps {
                echo "This is a build step"
            }
        }
        stage("test") {
            steps {
                echo "This is a test step"
            }
        }
        stage("odc-backend") {
            agent {
                docker { 
                    image 'docker:20.10-dind'
                    args '-u root -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                sh """
                chmod +x run-depcheck.sh
                ./run-depcheck.sh
                """
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reports/dependency-check-report.json', onlyIfSuccessful: false
                    }
                }
            }
        }
        stage("integration") {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    echo "This is an integration step."
                    sh "exit 1"
                }
            }
        }
        stage("prod") {
            steps {
                timeout(time: 10, unit: 'SECONDS') {
                    input "Deploy to production?"
                }
                echo "This is a deploy step."
            }
        }
    }
    post {
        failure {
            updateGitlabCommitStatus name: STAGE_NAME, state: 'failed'
        }
        unstable {
            updateGitlabCommitStatus name: STAGE_NAME, state: 'failed'
        }
        success {
            updateGitlabCommitStatus name: STAGE_NAME, state: 'success'
        }
        always {
            archiveArtifacts artifacts: '*.json', fingerprint: true
            deleteDir()                     // clean up workspace
            dir("${WORKSPACE}@tmp") {       // clean up tmp directory
                deleteDir()
            }
            dir("${WORKSPACE}@script") {    // clean up script directory
                deleteDir()
            }
        }
    }
}
