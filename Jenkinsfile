pipeline {
    agent any
    stages {
        stage('checkout form GITHUB') {
            steps {
                git url: 'https://github.com/ashishsarkar/sonarqube-setup.git'
            }
        }
    

        stage('Build && Scanner') {
            steps {
                git url: 'https://github.com/ashishsarkar/sonarqube-setup.git'
            }
        }
}
}