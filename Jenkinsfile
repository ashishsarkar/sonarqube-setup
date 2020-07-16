pipeline {
    agent any
    options { 
        buildDiscarder(logRotator(numToKeepStr: "10"))
        timeout(time: 30, unit: 'MINUTES')
    }    
    triggers {
        pollSCM '* * * * *'
    }
    environment
    {
        VERSION = "1.0.2_${BUILD_NUMBER}"
        PROJECT = 'nodeapp'
        IMAGE = "$PROJECT:$VERSION"
        DEV_ECR_URI = '106102357433.dkr.ecr.ap-south-1.amazonaws.com'
        ECRURL = 'https://106102357433.dkr.ecr.ap-south-1.amazonaws.com'
        // ECRCRED = 'ecr:ap-south-1:AWS_ECR_credentials'
        COMMITID = sh(script: 'git rev-parse --short HEAD', returnStdout: true)
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
        stage('Build preparations') {
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
                    // currentBuild.displayName = "#" + "1.v" + env.BUILD_NUMBER + " " + params.action + " AWS-ECR-" + params.action
                    IMAGE = "$PROJECT:$VERSION"
                }
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
        stage('Build Image using Docker') {
            steps
            {
               
                    echo "Build Image using Docker..................."

                    // Build the docker image using a Dockerfile
                   

                    sh "docker build -t nodeapp ."
                    sh "docker tag nodeapp:latest 106102357433.dkr.ecr.ap-south-1.amazonaws.com/nodeapp:v2$BUILD_ID$VERSION"
                     echo "Build Image using Docker  Completed..................."
                
            }
        }

        stage('Push Image to ECR') {
              steps
                {
                    script
                    {
                    
                    // Push the Docker image to ECR
                    // docker.withRegistry(ECRURL, ECRCRED)
                    // {
                    //     docker.image(IMAGE).push()
                    // }

                    sh "docker push 106102357433.dkr.ecr.ap-south-1.amazonaws.com/nodeapp:v2$VERSION"
                    echo "Validation completed................"
                }                    
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
