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

        // stage('BUILD') {
        //     steps {
        //         sh 'sudo apt install nodejs -y'
        //         sh 'sudo apt install npm -y'
        //         sh 'npm install'
        //         sh 'npm audit fix --force'
        //     }
        // }

        stage('Run SonarCloud Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner 4.0'
                    withSonarQubeEnv(credentialsId: SONAR_TOKEN, installationName: 'sonar-server') {
                        // Run SonarCloud analysis
                        
                        sh "${scannerHome}/bin/sonar-scanner"
                        sonar-scanner(
                            '-Dsonar.organization=greyabiwon-projects',
                            '-Dsonar.projectKey=greyabiwon-projects_deckmaster',
                            '-Dsonar.sources=.',
                            '-Dsonar.host.url=https://sonarcloud.io'
                        )
                        timeout(time: 10, unit: 'MINUTES') {
                            waitForQualityGate abortPipeline: true
                        }
                    }
                }
            }
        }
    }

    // stage('Building image') {
    //     steps {
    //         script {
    //             def dockerImageName = "${registry}:${BUILD_NUMBER}"
    //             dockerImage = docker.build dockerImageName
    //         }
    //     }
    // }

    // stage('Trivy Scan') {
    //     steps {
    //         script {
    //             // Define the Docker image name and tag (replace with your actual image name and tag)
    //             def dockerImageName = "${registry}:${BUILD_NUMBER}"

    //             // Run Trivy scan on your Docker image
    //             def trivyScanResult = sh(script: "trivy image ${dockerImageName}", returnStatus: true)

    //             if (trivyScanResult == 0) {
    //                 echo 'Trivy scan passed. No vulnerabilities found.'
    //             } else {
    //                 error 'Trivy scan failed. Vulnerabilities detected.'
    //             }
    //         }
    //     }
    // }

    // stage('Deploy Image') {
    //     steps {
    //         script {
    //             docker.withRegistry('', registryCredential) {
    //                 dockerImage.push("latest")
    //             }
    //         }
    //     }
    // }

    // stage('Remove Unused docker image') {
    //     steps {
    //         sh "docker rmi ${registry}:${BUILD_NUMBER}"
    //     }
    // }

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
