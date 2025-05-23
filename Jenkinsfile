pipeline {
    agent any

    environment {
        DOTNET_PATH = 'C:\\Program Files\\dotnet'
        DOCKER_PATH = 'C:\\Program Files\\Docker\\Docker\\resources\\bin'
        TERRAFORM_PATH = 'C:\\Users\\window 10\\Downloads\\terraform_1.11.3_windows_386'
        AZURE_CLI_PATH = 'C:\\Program Files\\Microsoft SDKs\\Azure\\CLI2\\wbin'
        
        PATH = "${DOTNET_PATH};${DOCKER_PATH};${TERRAFORM_PATH};${AZURE_CLI_PATH};${PATH}"
        
        ACR_NAME = 'kubernetesacr04082003'
        AZURE_CREDENTIALS_ID = 'azure-service-principal-kubernetes'
        ACR_LOGIN_SERVER = "${ACR_NAME}.azurecr.io"
        IMAGE_NAME = 'kubernetes04082003'
        IMAGE_TAG = 'latest'
        RESOURCE_GROUP = 'rg-aks-acr'
        AKS_CLUSTER = 'mycluster04082003'
        TF_WORKING_DIR = 'Terraform'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/aditya-blanko/Terraform-Jenkins-Docker-Dotnet.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %ACR_LOGIN_SERVER%/%IMAGE_NAME%:%IMAGE_TAG% -f DotNetWebAPP/Dockerfile DotNetWebAPP"
            }
        }

       stage('Terraform Init') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat """
                    az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID
                    cd %TF_WORKING_DIR%
                    terraform init
                    """
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat """
                    cd %TF_WORKING_DIR%
                    terraform plan
                    """
                }
            }
        }


        stage('Terraform Apply') {
    steps {
        withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
            bat """
            cd %TF_WORKING_DIR%
            terraform apply -auto-approve 
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
                bat "kubectl apply -f deployment.yml"
                bat "kubectl get service dotnet-api-service" 
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
