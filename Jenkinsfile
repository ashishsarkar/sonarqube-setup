pipeline {
    agent any
    environment
    {
        VERSION = "1.0.2_${BUILD_NUMBER}"
        PROJECT = 'nodeapp'
        IMAGE = "$PROJECT:$VERSION"
        ECRURL = 'https://106102357433.dkr.ecr.ap-south-1.amazonaws.com'
        ECRCRED = 'ecr:ap-south-1:AWS_ECR_credentials'
    }
    stages {
        stage('checkout from SCM') {
            steps {
                echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
                echo "Build Number -- $VERSION"
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

        stage("ECR Login") {
            steps {
                echo "ECR Login  process started..."
                withAWS(credentials:'AWS_ECR_credentials') {
                    script {
                        login = aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 106102357433.dkr.ecr.ap-south-1.amazonaws.com
                        sh "${login}"
                    }
                }
                echo "ECR Login process Completed..."
            }
        }



        // stage('Image Build') {
        //       steps {
        //           echo "Image Build process started..."
        //           script{
        //                docker.build('$IMAGE')
        //         }
        //         echo "Image Build process Completed..."
        //     }
        // }
        // stage('Push Image') {
        //       steps {
        //           echo "Push Image process started..."
        //           script{
        //                docker.withRegistry(ECRURL, ECRCRED)
        //                {
        //                    docker.image(IMAGE).push()
        //                }
        //         }
        //     }
        // }
    }


    // post
    //     {
    //         always
    //         {
    //             // make sure that the Docker image is removed
    //             sh "docker rmi $IMAGE | true"
    //         }
    //     }
}