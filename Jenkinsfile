pipeline {
    agent any

    environment {
        SONAR_SCANNER = "/opt/sonar-scanner/bin/sonar-scanner"  // Correct sonar-scanner path
        SONAR_HOST_URL = "http://your-sonarqube-server:9000"  // SonarQube server URL (Remote)
        SONAR_PROJECT_KEY = "example-repo"                     // Replace with your actual SonarQube project key
        SONARQUBE_TOKEN = "squ_1cd7bff50570147edc00da9e1bad2d77591d2085"  // SonarQube authentication token
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
                    // Run SonarQube scan
                    sh """
                        ${env.SONAR_SCANNER} \
                        -Dsonar.projectKey=${env.SONAR_PROJECT_KEY} \
                        -Dsonar.host.url=${env.SONAR_HOST_URL} \
                        -Dsonar.login=${env.SONARQUBE_TOKEN}
                    """
                }
            }
        }

        stage('Testing') {
            steps {
                // Run your tests (adjust according to your build tool, e.g., Maven, Gradle)
                sh 'mvn test'  // Replace with your specific test command if needed
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    echo "⏳ Waiting for SonarQube analysis to complete..."

                    def qualityGateStatus = ""
                    for (int i = 0; i < 5; i++) {  // Retry for up to 50 seconds
                        def sonarResponse = sh(script: """
                            curl -s -u ${env.SONARQUBE_TOKEN}: ${env.SONAR_HOST_URL}/api/qualitygates/project_status?projectKey=${env.SONAR_PROJECT_KEY}
                        """, returnStdout: true).trim()
                        
                        echo "SonarQube Response: ${sonarResponse}"
                        
                        qualityGateStatus = sh(script: "echo '${sonarResponse}' | jq -r '.projectStatus.status'", returnStdout: true).trim()
                        
                        if (qualityGateStatus == "OK") {
                            echo "✅ Quality Gate passed!"
                            break
                        } else if (qualityGateStatus == "ERROR") {
                            error "❌ Quality Gate failed! Fix issues before proceeding."
                        }
                        
                        echo "🔄 Retrying in 10 seconds..."
                        sleep(10)
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
