pipeline {

    agent {
        node {
            label 'AGENT-1'
        }
    }

    environment {
        COURSE      = "Jenkins"
        appVersion  = ""
        ACC_ID      = "526426842890"
        PROJECT     = "roboshop"
        COMPONENT   = "catalogue"
        REGION      = "us-east-1"

        ECR_REPO    = "${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com/${PROJECT}/${COMPONENT}"
    }


    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Read Version') {
            steps {
                script {
                    def packageJSON = readJSON file: 'package.json'
                    appVersion = packageJSON.version
                    echo "Application Version : ${appVersion}"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    npm install
                '''
            }
        }

        stage('Unit Test') {
            steps {
                sh '''
                    npm test
                '''
            }
        }

        stage('SonarQube Scan') {
            steps {
                script {

                    def scannerHome = tool 'SonarScanner'

                    withSonarQubeEnv('sonar-qube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                              -Dsonar.projectKey=${COMPONENT} \
                              -Dsonar.projectName=${COMPONENT} \
                              -Dsonar.sources=. \
                              -Dsonar.projectVersion=${appVersion}
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                    docker build -t ${PROJECT}/${COMPONENT}:${appVersion} .
                """
            }
        }

        /*
        stage('Trivy Image Scan') {
            steps {
                sh """
                    echo "=========================================="
                    echo "      Trivy Image Vulnerability Scan"
                    echo "=========================================="

                    docker images

                    trivy image \
                      --scanners vuln \
                      --severity LOW,MEDIUM,HIGH,CRITICAL \
                      --format table \
                      --ignore-unfixed \
                      localhost/${PROJECT}/${COMPONENT}:${appVersion}
                """
            }
        }
        */

        stage('Login to AWS ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-cred'
                ]]) {

                    sh """
                        aws ecr get-login-password --region ${REGION} | \
                        docker login \
                          --username AWS \
                          --password-stdin \
                          ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com
                    """
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh """
                    docker tag ${PROJECT}/${COMPONENT}:${appVersion} ${ECR_REPO}:${appVersion}
                    docker tag ${PROJECT}/${COMPONENT}:${appVersion} ${ECR_REPO}:latest
                """
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh """
                    docker push ${ECR_REPO}:${appVersion}
                    docker push ${ECR_REPO}:latest
                """
            }
        }

                stage('Terraform VPC') {
            steps {
                dir('/home/ec2-user/eks-automation-deployment/00-vpc') {

                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-cred'
                    ]]) {

                        sh '''
                            terraform init
                            terraform validate
                            terraform plan
                            terraform apply -auto-approve
                        '''
                    }
                }
            }
        }

        stage('Terraform Security Groups') {
            steps {
                dir('/home/ec2-user/eks-automation-deployment/10-sg') {

                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-cred'
                    ]]) {

                        sh '''
                            terraform init
                            terraform validate
                            terraform plan
                            terraform apply -auto-approve
                        '''
                    }
                }
            }
        }

        stage('Terraform Bastion') {
            steps {
                dir('/home/ec2-user/eks-automation-deployment/20-bastion') {

                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-cred'
                    ]]) {

                        sh '''
                            terraform init
                            terraform validate
                            terraform plan
                            terraform apply -auto-approve
                        '''
                    }
                }
            }
        }

        stage('Terraform SG Rules') {
            steps {
                dir('/home/ec2-user/eks-automation-deployment/30-sg-rules') {

                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-cred'
                    ]]) {

                        sh '''
                            terraform init
                            terraform validate
                            terraform plan
                            terraform apply -auto-approve
                        '''
                    }
                }
            }
        }

        stage('Terraform ACM') {
            steps {
                dir('/home/ec2-user/eks-automation-deployment/70-acm') {

                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-cred'
                    ]]) {

                        sh '''
                            terraform init
                            terraform validate
                            terraform plan
                            terraform apply -auto-approve
                        '''
                    }
                }
            }
        }

        stage('Terraform Frontend ALB') {
            steps {
                dir('/home/ec2-user/eks-automation-deployment/80-frontend-alb') {

                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-cred'
                    ]]) {

                        sh '''
                            terraform init
                            terraform validate
                            terraform plan
                            terraform apply -auto-approve
                        '''
                    }
                }
            }
        }

         stage('Terraform EKS') {
            steps {
                dir('/home/ec2-user/eks-automation-deployment/90-eks') {

                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-cred'
                    ]]) {

                        sh '''
                            terraform init
                            terraform validate
                            terraform plan
                            terraform apply -auto-approve
                        '''
                    }
                }
            }
        }

    }   // <-- Close stages block

    post {

        always {
            echo "Cleaning Workspace..."
            cleanWs()
        }

        success {
            echo "Pipeline Completed Successfully"
        }

        failure {
            echo "Pipeline Failed"
        }

        aborted {
            echo "Pipeline Aborted"
        }
    }

}