pipeline {
    agent {
        docker {
            image 'maven:3.9.9'
            args '-v /root/.m2:/root/.m2'
        }
    }
    triggers {
        pollSCM('*/2 * * * *') // Poll SCM every 2 minutes
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
        stage('Deploy') {
            steps {
                sh './jenkins/scripts/deliver.sh'
                sleep(time: 60, unit: 'SECONDS') // Wait for 30 seconds
                echo 'Service should be ready now.'
            }
        }
    }
}
