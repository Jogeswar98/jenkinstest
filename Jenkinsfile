pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Jogeswar98/jenkinstest.git'
            }
        }

        stage('Code Analysis') {
            steps {
                script {
                    sh 'sonar-scanner'
                }
            }
        }

        stage('Security Scan') {
            steps {
                sh 'dependency-check.sh --scan ./ --format HTML || true' // Prevent pipeline failure
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
                    def sonarStatus = sh(script: "curl -s -u YOUR_SONARQUBE_TOKEN: http://your-sonarqube-server-ip:9000/api/qualitygates/project_status?projectKey=example-repo", returnStdout: true)
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
