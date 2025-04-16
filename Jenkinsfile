pipeline {
    agent any

    environment {
        ACR_NAME = 'myacrnamemuskan'
        AZURE_CREDENTIALS_ID = 'azure-service-principal-1'
        ACR_LOGIN_SERVER = "${ACR_NAME}.azurecr.io"
        IMAGE_NAME = 'mywebapi'
        IMAGE_TAG = 'latest'
        RESOURCE_GROUP = 'myResourceGroup'
        AKS_CLUSTER = 'myAKSCluster'
        TF_WORKING_DIR = 'terraform'
        // Use dynamic way of finding Terraform binary, Jenkins tools can be used instead of hardcoding paths.
        TERRAFORM_PATH = tool name: 'terraform', type: 'Tool'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/MuskanFalwaria/aks-tf-jenkins.git'
            }
        }
        
        stage('Build .NET Web API') {
            steps {
                bat 'dotnet publish ApiContainer/ApiContainer.csproj -c Release -o out'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build --pull --progress=plain -t %ACR_LOGIN_SERVER%/%IMAGE_NAME%:%IMAGE_TAG% -f ApiContainer/Dockerfile .'
            }
        }

        stage('Terraform Init') {
            steps {
                bat '"%TERRAFORM_PATH%" -chdir=%TF_WORKING_DIR% init'
            }
        }

        stage('Terraform Plan') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat '"%TERRAFORM_PATH%" -chdir=%TF_WORKING_DIR% plan -out=tfplan'
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat '"%TERRAFORM_PATH%" -chdir=%TF_WORKING_DIR% apply -auto-approve tfplan'
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
            echo '✅ Deployment completed successfully!'
        }
        failure {
            echo '❌ Deployment failed. Check the logs.'
        }
    }
}


// pipeline {
//     agent any

//     environment {
//         ACR_NAME = 'myacrnamemuskan'
//         AZURE_CREDENTIALS_ID = 'azure-service-principal-1'
//         ACR_LOGIN_SERVER = "${ACR_NAME}.azurecr.io"
//         IMAGE_NAME = 'mywebapi'
//         IMAGE_TAG = 'latest'
//         RESOURCE_GROUP = 'myResourceGroup'
//         AKS_CLUSTER = 'myAKSCluster'
//         TF_WORKING_DIR = 'terraform'
//         TERRAFORM_PATH = '"C:\\Users\\ASUS\\Downloads\\terraform_1.11.3_windows_amd64\\terraform.exe"'
//     }

//     stages {
//         stage('Clean Workspace') {
//     steps {
//         cleanWs()
//     }
// }
//         stage('Checkout') {
//             steps {
//                 git branch: 'main', url: 'https://github.com/MuskanFalwaria/aks-tf-jenkins.git'
//             }
//         }
//         stage('Build .NET Web API') {
//             steps {
//                 bat 'dotnet publish ApiContainer/ApiContainer.csproj -c Release -o out'
//             }
//         }

//         stage('Build Docker Image') {
//             steps {
//                 bat 'docker build --pull --progress=plain -t %ACR_LOGIN_SERVER%/%IMAGE_NAME%:%IMAGE_TAG% -f ApiContainer/Dockerfile .'
//             }
//         }

//         stage('Terraform Init') {
//             steps {
//                 bat '"%TERRAFORM_PATH%" -chdir=%TF_WORKING_DIR% init'
//             }
//         }

// stage('Terraform Plan') {
//     steps {
//         withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
//             bat '"%TERRAFORM_PATH%" -chdir=%TF_WORKING_DIR% plan -out=tfplan'
//         }
//     }
// }

// stage('Terraform Apply') {
//     steps {
//         withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
//             bat '"%TERRAFORM_PATH%" -chdir=%TF_WORKING_DIR% apply -auto-approve tfplan'
//         }
//     }
// }


        


//         stage('Login to ACR') {
//             steps {
//                 bat "az acr login --name %ACR_NAME%"
//             }
//         }

//         stage('Push Docker Image to ACR') {
//             steps {
//                 bat "docker push %ACR_LOGIN_SERVER%/%IMAGE_NAME%:%IMAGE_TAG%"
//             }
//         }

//         stage('Get AKS Credentials') {
//             steps {
//                 bat "az aks get-credentials --resource-group %RESOURCE_GROUP% --name %AKS_CLUSTER% --overwrite-existing"
//             }
//         }

//         stage('Deploy to AKS') {
//             steps {
//                 bat "kubectl apply -f ApiContainer/deployment.yaml"
//             }
//         }
//     }

//     post {
//         success {
//             echo '✅ Deployment completed successfully!'
//         }
//         failure {
//             echo '❌ Deployment failed. Check the logs.'
//         }
//     }
// }

