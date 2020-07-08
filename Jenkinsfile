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
            steps
            {
                script 
                {
                    // calculate GIT lastest commit short-hash
                    gitCommitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    shortCommitHash = gitCommitHash.take(7)
                    // calculate a sample version tag
                    VERSION = shortCommitHash
                    // set the build display name
                    currentBuild.displayName = "#${BUILD_ID}-${VERSION}"
                    IMAGE = "$PROJECT:$VERSION"
                }
            }
        }




        stage('Push Image') {
              steps
                {
                    echo "Logging IN now..................."
                    script
                    {
                    // login to ECR - for now it seems that that the ECR Jenkins plugin is not performing the login as expected. I hope it will in the future.
                    sh("eval \$(aws ecr get-login --no-include-email | sed 's|https://||')")
                    // Push the Docker image to ECR
                    docker.withRegistry(ECRURL, ECRCRED)
                    {
                        docker.image(IMAGE).push()
                    }
                }

                    echo "Validation completed................"
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