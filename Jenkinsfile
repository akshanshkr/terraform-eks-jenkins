pipeline {
    agent any

    parameters {
            booleanParam(name: 'PLAN_TERRAFORM', defaultValue: false, description: 'Check to plan Terraform changes')
            booleanParam(name: 'APPLY_TERRAFORM', defaultValue: false, description: 'Check to apply Terraform changes')
            booleanParam(name: 'DESTROY_TERRAFORM', defaultValue: false, description: 'Check to apply Terraform changes')
            booleanParam(name: 'ADD_PYTHON_PROJECT', defaultValue: false, description: 'Check to apply python_project')

    }

    stages {
        stage('Clone Repository') {
            steps {
                // Clean workspace before cloning (optional)
                deleteDir()

                // Clone the Git repository
                git branch: 'main',
                    url : 'https://github.com/akshanshkr/terraform-eks-jenkins.git'

                sh "ls -lart"
            }
        }

        stage('Terraform Init') {
                    steps {
                       withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]){
                            dir('terraform') {
                            sh 'echo "=================Terraform Init=================="'
                            sh 'terraform init'
                        }
                    }
                }
        }

        stage('Terraform Plan') {
            steps {
                script {
                    if (params.PLAN_TERRAFORM) {
                       withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]){
                            dir('terraform') {
                                sh 'echo "=================Terraform Plan=================="'
                                sh 'terraform plan'
                            }
                        }
                    }
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                script {
                    if (params.APPLY_TERRAFORM) {
                       withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]){
                            dir('terraform') {
                                sh 'echo "=================Terraform Apply=================="'
                                sh 'terraform apply -auto-approve'
                            }
                        }
                    }
                }
            }
        }

        stage('Terraform Destroy') {
            steps {
                script {
                    if (params.DESTROY_TERRAFORM) {
                       withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]){
                            dir('terraform') {
                                sh 'echo "=================Terraform Destroy=================="'
                                sh 'terraform destroy -auto-approve'
                            }
                        }
                    }
                }
            }
        }

        stage('Check and Install AWS CLI') {
            steps {
                script {
                    // Check if aws CLI is installed
                    def awsCliInstalled = sh(script: 'which aws', returnStatus: true) == 0

                    if (!awsCliInstalled) {
                        echo "AWS CLI not found. Installing AWS CLI..."

                        // Install necessary packages
                        sh 'sudo apt-get update'
                        sh 'sudo apt-get install -y curl unzip'

                        // Download and install AWS CLI
                        sh '''
                        curl "https://d1uj6qtbmh3dt5.cloudfront.net/awscli-exe-linux-x86_64.zip" -o "awscli-exe-linux-x86_64.zip"
                        unzip awscli-exe-linux-x86_64.zip
                        sudo ./aws/install
                        rm -rf awscli-exe-linux-x86_64.zip aws
                        '''

                        // Verify installation
                        sh 'aws --version'
                    } else {
                        echo "AWS CLI is already installed."
                    }
                }
            }
        }

        stage('Check and Install kubectl') {
            steps {
                script {
                    // Check if kubectl is installed
                    def kubectlInstalled = sh(script: 'which kubectl', returnStatus: true) == 0

                    if (!kubectlInstalled) {
                        echo "kubectl not found. Installing kubectl..."

                        // Install necessary packages
                        sh 'sudo apt-get update'
                        sh 'sudo apt-get install -y curl apt-transport-https'

                        // Download and install kubectl
                        sh '''
                        curl -LO "https://dl.k8s.io/release/v1.26.0/bin/linux/amd64/kubectl"
                        chmod +x ./kubectl
                        sudo mv ./kubectl /usr/local/bin/kubectl
                        '''

                        // Verify installation
                        sh 'kubectl version --client'
                    } else {
                        echo "kubectl is already installed."
                    }
                }
            }
        }

        stage('kubectl install') {
            steps {
                script {
                    if (params.ADD_PYTHON_PROJECT) {
                       withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]){
                            dir('terraform') {
                                sh 'echo "=================connecting kubectl with aws =================="'
                                // aws eks update-kubeconfig --region ${env.AWS_REGION} --name ${env.EKS_CLUSTER_NAME}
                                sh 'aws eks update-kubeconfig --region us-east-2 --name education-eks-Qg7sWMk5'
                            }
                        }
                    }
                }
            }
        }

        stage('add-python-project') {
            steps {
                script {
                    if (params.ADD_PYTHON_PROJECT) {
                       withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]){
                            dir('application') {
                                sh 'echo "=================installing application to run=================="'
                                sh 'kubectl apply -f mysql-configmap.yml'
                                sh 'kubectl apply -f mysql-secrets.yml'
                                sh 'kubectl apply -f mysql-deployment.yml'
                                sh 'kubectl apply -f mysql-svc.yml'
                                sh 'kubectl apply -f two-tier-app-deployment.yml'
                                sh 'kubectl apply -f two-tier-app-svc.yml'
                                
                            }
                        }
                    }
                }
            }
        }
    }
}