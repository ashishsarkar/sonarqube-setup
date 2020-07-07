pipeline {
    agent any
    stages {
        stage('checkout form SCM') {
            steps {
                echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
                echo "Build Number ${env.BUILD_NUMBER}"
                echo "Build Tag ${env.BUILD_TAG}"
                echo "Build URL ${env.BUILD_URL}"
                echo "Build Executor Number ${env.EXECUTOR_NUMBER}"
                echo "Build JAVA HOME ${env.JAVA_HOME}"
                echo "Build JOB NAME ${env.JOB_NAME}"
                echo "Build Node Name ${env.NODE_NAME}"
                echo "Build WORKSPACE ${env.WORKSPACE}"
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