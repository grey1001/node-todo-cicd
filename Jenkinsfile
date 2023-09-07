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

        stage('Run SonarCloud Analysis') {
            steps {
                script {
                    // Define the SonarScanner tool installation
                    def scannerHome = tool 'sonar-server'
                    
                    // Set environment variables for SonarQube credentials
                    withSonarQubeEnv(credentialsId: 'SONAR_TOKEN', installationName: 'sonar-server') {
                        // Run SonarCloud analysis using the specified tool
                        sh "${scannerHome}/bin/sonar-scanner"
                        
                        // Optional: You can specify additional analysis parameters here
                        // sh "${scannerHome}/bin/sonar-scanner -Dsonar.parameter=value"
                        
                        // Wait for the quality gate and abort the pipeline if it fails
                        timeout(time: 10, unit: 'MINUTES') {
                            waitForQualityGate abortPipeline: true
                        }
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
