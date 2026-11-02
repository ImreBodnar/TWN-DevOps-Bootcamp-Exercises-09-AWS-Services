#!/user/bin/env groovy

@Library('TWN-Jenkins-Shared-Library')_

pipeline {
    agent any

    tools {
        nodejs "my-nodejs"
    }

    environment {
        NEXUS_ADDRESS = "161.35.197.245:8082"
    }

    stages {
        stage("IncrementVersion") {
            steps {
                script {
                    echo "----- INCREMENTING VERSION -----"

                    dir("app") {
                        incrementVersion()
                    }
                }
            }
        }

        stage("RunTests") {
            steps {
                script {
                    echo "----- RUNNING TESTS -----"

                    dir("app") {
                        runNpmTests()
                    }
                }
            }
        }

        stage("BuildImage") {
            steps {
                script {
                    echo "----- BUILDING DOCKER IMAGE -----"

                    buildDockerImage('nodejs-app-for-aws')
                }
            }
        }

        stage("PushImageToRegistry") {
            steps {
                script {
                    echo "----- PUSHING DOCKER IMAGE TO NEXUS -----"

                    pushDockerImageToRegistry('nodejs-app-for-aws')
                }
            }
        }

        stage("CommitNewVersion") {
            steps {
                script {
                    echo "----- COMMITING & PUSHING NEW VERSION TO GIT -----"

                    commitNewVersion('TWN-DevOps-Bootcamp-Exercises-09-AWS-Services.git')
                }
            }
        }
    }

    post {
        always {
            echo "END OF PIPELINE EXECUTION."
        }

        success {
            echo "THE PIPELINE FINISHED SUCCESSFULLY!"
        }

        failure {
            echo "SOME ERRORS OCCURED!"
        }
    }
}