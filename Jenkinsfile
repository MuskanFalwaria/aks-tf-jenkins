pipeline {
    agent any

    environment {
        ACR_NAME = 'myacrnamemuskan'
        AZURE_CREDENTIALS_ID = 'jenkins-pipeline-sp'
        ACR_LOGIN_SERVER = "${ACR_NAME}.azurecr.io"
        IMAGE_NAME = 'mywebapi'
        IMAGE_TAG = 'latest'
        RESOURCE_GROUP = 'myResourceGroup'
        AKS_CLUSTER = 'myAKSCluster'
        TF_WORKING_DIR = '.'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/MuskanFalwaria/aks-tf-jenkins.git'
            }
        }

        stage('Build .NET App') {
            steps {
                bat 'dotnet publish ApiContainer/ApiContainer.csproj -c Release -o out'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %ACR_LOGIN_SERVER%/%IMAGE_NAME%:%IMAGE_TAG% -f ApiContainer/Dockerfile ApiContainer"
            }
        }

       stage('Terraform Init') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat """
                    echo "Navigating to Terraform Directory: %TF_WORKING_DIR%"
                    cd %TF_WORKING_DIR%
                    echo "Initializing Terraform..."
                    terraform init
                    """
                }
            }
        }

        stage('Terraform Plan') {
    steps {
        withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
            bat """
            echo "Navigating to Terraform Directory: %TF_WORKING_DIR%"
            cd %TF_WORKING_DIR%
            terraform plan -out=tfplan
            """
        }
    }
}


        stage('Terraform Apply') {
    steps {
        withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
            bat """
            echo "Navigating to Terraform Directory: %TF_WORKING_DIR%"
            cd %TF_WORKING_DIR%
            echo "Applying Terraform Plan..."
            terraform apply -auto-approve tfplan
            """
        }
    }
}
        stage('Login to ACR') {
            steps {
                bat "az acr login --name %ACR_NAME%"
            }
        }

        stage('Push Docker Image to ACR') {
            steps {
                bat "docker push %ACR_LOGIN_SERVER%/%IMAGE_NAME%:%IMAGE_TAG%"
            }
        }

        stage('Get AKS Credentials') {
            steps {
                bat "az aks get-credentials --resource-group %RESOURCE_GROUP% --name %AKS_CLUSTER% --overwrite-existing"
            }
        }

        stage('Deploy to AKS') {
            steps {
                bat "kubectl apply -f ApiContainer/deployment.yaml"
            }
        }
    }

    post {
        success {
            echo 'All stages completed successfully!'
        }
        failure {
            echo 'Build failed.'
        }
    }
}
// pipeline {
//     agent any

//     environment {
//         ACR_NAME = 'myacrnamemuskan'
//         IMAGE_NAME = 'mywebapi'
//         RESOURCE_GROUP = 'myResourceGroup'
//         TERRAFORM_VERSION = '1.7.5'
//         TERRAFORM_DIR = "${env.WORKSPACE}\\terraform-bin"
//         PATH = "${env.TERRAFORM_DIR};${env.PATH}"
//     }

//     stages {

//         stage('Setup Terraform') {
//             steps {
//                 bat """
//                 if not exist %TERRAFORM_DIR% (
//                     mkdir %TERRAFORM_DIR%
//                 )
//                 curl -o terraform.zip https://releases.hashicorp.com/terraform/%TERRAFORM_VERSION%/terraform_%TERRAFORM_VERSION%_windows_amd64.zip
//                 powershell -Command "Expand-Archive -Path terraform.zip -DestinationPath %TERRAFORM_DIR% -Force"
//                 del terraform.zip
//                 terraform -v
//                 """
//             }
//         }

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
//                     bat 'docker login %ACR_NAME%.azurecr.io -u %USERNAME% -p %PASSWORD%'
//                     bat 'docker push %ACR_NAME%.azurecr.io/%IMAGE_NAME%'
//                 }
//             }
//         }

//         stage('Deploy to AKS') {
//             steps {
//                 script {
//                     bat 'az aks get-credentials --resource-group %RESOURCE_GROUP% --name myAKSCluster'
//                     bat 'kubectl apply -f deployment.yaml'
//                 }
//             }
//         }
//     }
// }

