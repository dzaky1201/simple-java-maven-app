pipeline {
    agent {
        docker {
            image 'maven:3.9.9'
            args '-v /root/.m2:/root/.m2'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Manual Approval') {
            steps {
                script {
                    // Pause the pipeline and wait for manual approval
                    def userInput = input(
                        id: 'userInput', 
                        message: 'Lanjutkan ke tahap Deploy?', 
                    )
                    // Check the user's input
                    if (userInput == 'Abort') {
                        error 'Deployment rejected by user.'
                    } else {
                        echo 'Deployment approved by user. Proceeding...'
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                // Use sshPublisher to upload the built artifacts to EC2
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: 'ssh-submission', // Name of the SSH server configuration in Jenkins
                            transfers: [
                                 sshTransfer(
                                    sourceFiles: 'target/*.jar', // Path to the built JAR file
                                    removePrefix: 'target', // Remove the 'target' prefix from the file path
                                    remoteDirectory: '/home/ubuntu/app', // Remote directory on EC2
                                    execCommand: """
                                        cd /home/ubuntu/app/app/home/ubuntu/app
                                        echo 'Starting new application...'
                                        java -jar my-app-1.0-SNAPSHOT.jar
                                    """
                                )
                            ],
                            usePromotionTimestamp: false,
                            useWorkspaceInPromotion: false,
                            verbose: true
                        )
                    ]
                )
                // sleep(time: 60, unit: 'SECONDS') // Wait for 30 seconds
            }
        }
    }
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
