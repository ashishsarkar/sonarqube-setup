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
                script {
                    withSonarQubeEnv('sonarqube') {
                    sh "${SCANNER_HOME}/bin/sonar-scanner"
                    }
                    timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                }
            }
        }
        

        stage('Docker Build & Push to Docker Hub') {
              steps {
                  script {
                        sh 'cp -r ../devops-training@2/target .'
                                      sh 'docker build . -t deekshithsn/devops-training:$Docker_tag'
                          withCredentials([string(credentialsId: 'docker_password', variable: 'docker_password')]) {
                                
                              sh 'docker login -u deekshithsn -p $docker_password'
                              sh 'docker push deekshithsn/devops-training:$Docker_tag'
                          }
                       }
                    }
                 }

    }
}