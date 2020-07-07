pipeline {
    agent any
    stages {
        stage('checkout form SCM') {
            steps {
                echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
                echo ""
                git url: 'https://github.com/ashishsarkar/sonarqube-setup.git'
            }
        }
        stage('Quality Gate Scanner') {
            environment {
                SCANNER_HOME = tool 'sonar_scanner'
            }
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh "${SCANNER_HOME}/bin/sonar-scanner"
                }
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}