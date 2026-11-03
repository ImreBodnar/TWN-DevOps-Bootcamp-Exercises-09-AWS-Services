#!/user/bin/env groovy

@Library('TWN-Jenkins-Shared-Library')_

pipeline {
    agent any

    tools {
        nodejs "my-nodejs"
    }

    environment {
        NEXUS_ADDRESS = "161.35.197.245:8082"
        APP_NAME = "nodejs-app-for-aws"
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

                    buildDockerImage(APP_NAME)
                }
            }
        }

        stage("PushImageToRegistry") {
            steps {
                script {
                    echo "----- PUSHING DOCKER IMAGE TO NEXUS -----"

                    pushDockerImageToRegistry(APP_NAME)
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

        stage("DeployToEC2") {
            steps {
                script {
                    echo "----- DEPLOYING TO AWS EC2 INSTANCE -----"

                    env.IMAGE_NAME = "${NEXUS_ADDRESS}/${APP_NAME}:${NEW_VERSION}"
                    echo "FULL IMAGE NAME: ${IMAGE_NAME}"

                    sshagent(['twn-exercise-9']) {
                        def ec2Instance = "ubuntu@16.171.231.89"
                        def shellCommand = "bash ./run-docker-compose.sh ${IMAGE_NAME}"

                        sh "scp -o StrictHostKeyChecking=no run-docker-compose.sh ${ec2Instance}:/home/ubuntu"
                        sh "scp -o StrictHostKeyChecking=no docker-compose.yaml ${ec2Instance}:/home/ubuntu"
                        sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCommand}"
                    }
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