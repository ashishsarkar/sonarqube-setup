pipeline {
   
    agent any
    
    options { 
        buildDiscarder(logRotator(numToKeepStr: "10"))
        timeout(time: 30, unit: 'MINUTES')
    }  

   
    triggers {
        pollSCM '* * * * *'
    }
    
    environment {

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
                    sh "docker build -t 106102357433.dkr.ecr.ap-south-1.amazonaws.com/nodeapp:v2$BUILD_ID$VERSION ."
                    echo "Build Image using Docker  Completed..................."
            }
        }



        stage('Push Image to ECR') {
              steps
                {
                    script
                    {
                     
                        sh "docker push 106102357433.dkr.ecr.ap-south-1.amazonaws.com/nodeapp:v2$BUILD_ID$VERSION"
                        echo "Validation completed................"
                    }                    
                }   
            post {
                always {
                    sh "docker rmi -f 106102357433.dkr.ecr.ap-south-1.amazonaws.com/nodeapp:v2$BUILD_ID$VERSION | true"
                }
            }
        }

        // stage('Deploying to Dev EKS') {
                // when {
                //     expression {
                //         return(env.BRANCH_NAME=="${env.DEV_BUILD_BRANCH}" && env.TIER == "dev")
                //     }
                // }
                // environment {
                //     IMAGE_NAME = "${env.DEV_ECR_URI}" + ":" + "${env.COMMIT_ID}"
                //     APP_DOMAIN_NAME = "${env.PROJECT_NAME}-${env.COMPONENT}.int.dev.affinionservices.com"
                //     KEYCLOAK_SERVER_URL = "https://keycloak-ha.dev.affinionservices.com"
                // }
            //     steps {
            //         echo "Deploying to Dev EKS"
            //         kubernetesDeploy(
            //             kubeconfigId: "dev_eks_config",
            //             configs: "kube.yaml",
            //             enableConfigSubstitution: true
            //         )
            //         echo "Passed 1st step..... to Dev EKS"

            //         timeout(time: 650, unit: 'SECONDS') {
            //             echo "Passed 2nd step..... to Dev EKS"
            //             //Waiting for deployment to rollout successfully
            //             // sh "kubectl rollout status --watch -n ${env.NAMESPACE} deployments ${env.PROJECT_NAME}-${env.COMPONENT}-deployment --kubeconfig ~/.kube/${env.TIER}-gce-nextgen-eks-config.yaml"
            //                //sh "kubectl rollout status --watch -n default --kubeconfig ~/.kube/config.yaml"
            //                sh 'kubectl create -f kube.yaml'
            //         echo "Passed 3rd step..... to Dev EKS"
            //         }
            //     }
            // }
    }

        post {
            always {
                echo "One way or another, I have finished"
                deleteDir() /* clean up our workspace */
            }
        }
    
}
