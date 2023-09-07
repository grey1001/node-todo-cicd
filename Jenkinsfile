pipeline {
    agent { label 'jenkins-slave' }

    environment {
        registry = "greyabiwon/nodejsmedium"
        registryCredential = 'docker-login'
        SONAR_TOKEN = 'SONAR_TOKEN' // Use the actual credential ID here
        SLACK_TOKEN = 'slack-token'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout your source code from your version control system
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonar-server'
                    withSonarQubeEnv() {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }
    }

    post {
        failure {
            slackSend(
                color: '#FF0000',
                message: "Pipeline failed: ${currentBuild.fullDisplayName}",
                tokenCredentialId: 'slack-token',
                channel: '#devops-cicd'
            )
        }
        success {
            slackSend(
                color: 'good',
                message: "Pipeline succeeded: ${currentBuild.fullDisplayName}",
                tokenCredentialId: SLACK_TOKEN,
                channel: '#devops-cicd'
            )
        }
    }
}
