pipeline {
    agent any

    environment {
        ACR_NAME = 'myacrnamemuskan'
        IMAGE_NAME = 'mywebapi'
        RESOURCE_GROUP = 'myResourceGroup'
        TERRAFORM_VERSION = '1.7.5'
        TERRAFORM_DIR = "${env.WORKSPACE}\\terraform-bin"
        PATH = "${env.TERRAFORM_DIR};${env.PATH}"
    }

    stages {

        stage('Setup Terraform') {
            steps {
                bat """
                if not exist %TERRAFORM_DIR% (
                    mkdir %TERRAFORM_DIR%
                )
                curl -o terraform.zip https://releases.hashicorp.com/terraform/%TERRAFORM_VERSION%/terraform_%TERRAFORM_VERSION%_windows_amd64.zip
                powershell -Command "Expand-Archive -Path terraform.zip -DestinationPath %TERRAFORM_DIR% -Force"
                del terraform.zip
                terraform -v
                """
            }
        }

        stage('Terraform Init & Apply') {
            steps {
                dir('terraform') {
                    bat 'terraform init'
                    bat 'terraform apply -auto-approve'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${ACR_NAME}.azurecr.io/${IMAGE_NAME}")
                }
            }
        }

        stage('Push Docker Image to ACR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'acr-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    bat 'docker login %ACR_NAME%.azurecr.io -u %USERNAME% -p %PASSWORD%'
                    bat 'docker push %ACR_NAME%.azurecr.io/%IMAGE_NAME%'
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                script {
                    bat 'az aks get-credentials --resource-group %RESOURCE_GROUP% --name myAKSCluster'
                    bat 'kubectl apply -f deployment.yaml'
                }
            }
        }
    }
}

// pipeline {
//     agent any

//     environment {
//         ACR_NAME = 'myacrnamemuskan'
//         IMAGE_NAME = 'mywebapi'
//         RESOURCE_GROUP = 'myResourceGroup'
//     }

//     stages {
//         stage('Terraform Init & Apply') {
//             steps {
//                 dir('terraform') {
//                     bat 'terraform init'
//                     bat 'terraform apply -auto-approve'
//                 }
//             }
//         }

//         stage('Build Docker Image') {
//             steps {
//                 script {
//                     dockerImage = docker.build("${ACR_NAME}.azurecr.io/${IMAGE_NAME}")
//                 }
//             }
//         }

//         stage('Push Docker Image to ACR') {
//             steps {
//                 withCredentials([usernamePassword(credentialsId: 'acr-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
//                     bat 'docker login ${ACR_NAME}.azurecr.io -u $USERNAME -p $PASSWORD'
//                     bat 'docker push ${ACR_NAME}.azurecr.io/${IMAGE_NAME}'
//                 }
//             }
//         }

//         stage('Deploy to AKS') {
//             steps {
//                 script {
//                     bat 'az aks get-credentials --resource-group $RESOURCE_GROUP --name myAKSCluster'
//                     bat 'kubectl apply -f deployment.yaml'
//                 }
//             }
//         }
//     }
// }
