pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/example-repo.git'
            }
        }

        stage('Code Analysis') {
            steps {
                script {
                    def sonarHome = tool name: 'SonarQube Scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    sh "${sonarHome}/bin/sonar-scanner"
                }
            }
        }

        stage('Security Scan') {
            steps {
                sh 'dependency-check.sh --scan ./ --format HTML'
            }
        }

        stage('Testing') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    def sonarStatus = sh(script: "curl -u admin:admin http://your-sonarqube-server-ip:9000/api/qualitygates/project_status?projectKey=example-repo", returnStdout: true)
                    if (sonarStatus.contains('"status":"ERROR"')) {
                        error "Quality Gate failed! Fix issues before proceeding."
                    }
                }
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed due to a quality gate failure!'
        }
    }
}
