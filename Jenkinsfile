//file:noinspection GroovyAssignabilityCheck
pipeline {

    agent { label 'RaspberryPi_MasterNode' }
    tools {
        maven 'Maven 3.6.3 Rasp'
        jdk 'Java11 (openjdk) ARM'
    }

    options { buildDiscarder(logRotator(numToKeepStr: '3', artifactNumToKeepStr: '1')) }


    stages {

        stage("MVN Clean") {
            steps {
                withMaven {
                    sh "mvn clean"
                }
            }
        }
        stage("Compile source code") {
            steps {
                withMaven {
                    sh "mvn compile -P prod"
                }
            }
        }

        stage("Packaging SpringBoot artifact") {
            steps {
                withMaven {
                    sh 'mvn jar:jar spring-boot:repackage -P prod'
                }
            }
        }
        stage("Delivering artifact to AWS") {
            steps {
                echo 'Delivering artifact to remote server...'
                sshagent(['jenkinsAWSssh']) {
                    sh 'scp -v -o "StrictHostKeyChecking=no" ./target/*.jar ubuntu@enumerable-entity.host:/home/ubuntu/app/easymenu'
                }
            }
        }
        stage("SpringBoot runner restarting") {
            steps {
                script {
                    echo 'Deploying...'
                    sshagent(credentials: ['jenkinsAWSssh']) {
                        sh """ ssh ubuntu@enumerable-entity.host << EOF
                                          cd app/easymenu
                                          docker-compose restart
                                          exit
                                          EOF
                            """
                    }
                }
            }
        }
    }
}