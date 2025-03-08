pipeline {
    agent any

    environment {
        SONAR_SCANNER = "/opt/sonar-scanner/bin/sonar-scanner"  // Ensure Jenkins uses correct SonarScanner path
        SONAR_HOST_URL = "http://your-sonarqube-server:9000"     // Replace with your SonarQube server IP/domain
        SONAR_PROJECT_KEY = "example-repo"                      // Replace with your actual project key
        SONARQUBE_TOKEN = "your-sonarqube-token"                // Replace with your actual SonarQube token
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Jogeswar98/jenkinstest.git'
            }
        }

        stage('Code Analysis') {
            steps {
                script {
                    sh '$SONAR_SCANNER -Dsonar.projectKey=$SONAR_PROJECT_KEY -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=$SONARQUBE_TOKEN'
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
                    sleep(10)  // Give SonarQube time to analyze results
                    def sonarStatus = sh(script: "curl -s -u $SONARQUBE_TOKEN: $SONAR_HOST_URL/api/qualitygates/project_status?projectKey=$SONAR_PROJECT_KEY", returnStdout: true).trim()
                    echo "SonarQube Response: $sonarStatus"
                    if (sonarStatus.contains('"status":"ERROR"')) {
                        error "❌ Quality Gate failed! Fix issues before proceeding."
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed due to a quality gate or other issue!'
        }
    }
}
