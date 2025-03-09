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
                        parameters: [
                            choice(
                                name: 'approval', 
                                choices: ['Proceed', 'Abort'], 
                                description: 'Choose to approve or reject the deployment'
                            )
                        ]
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
                sh './jenkins/scripts/deliver.sh'
                sleep(time: 60, unit: 'SECONDS') // Wait for 30 seconds
                sh './jenkins/scripts/kill.sh'
            }
        }
    }
}
