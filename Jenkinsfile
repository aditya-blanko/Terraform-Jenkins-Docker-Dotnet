pipeline {
    agent any

    environment {
        ACR_NAME            = 'kubernetes-acr-04082003'
        ACR_LOGIN_SERVER    = "kubernetes-acr-04082003.azurecr.io"
        IMAGE_NAME          = 'kubernetes04082003'
        RESOURCE_GROUP      = 'rg-aks-acr'
        AKS_CLUSTER         = 'my-cluster-04082003'
        AZURE_CREDENTIALS_ID = 'azure-service-principal-kubernetes'
    }

    stages {
        stage('Azure Login') {
            steps {
                bat 'az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%'
                bat 'az account set --subscription %AZURE_SUBSCRIPTION%'
            }
        }

        stage('Terraform Init & Apply') {
            steps {
                dir('terraform') {
                    bat 'terraform init'
                    bat 'terraform plan'
                    bat 'terraform apply -auto-approve'
                }
            }
        }

        stage('Login to ACR') {
            steps {
                bat 'az acr login --name %ACR_NAME%'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %ACR_LOGIN_SERVER%/%IMAGE_NAME%:latest .'
            }
        }

        stage('Push Image to ACR') {
            steps {
                bat 'docker push %ACR_LOGIN_SERVER%/%IMAGE_NAME%:latest'
            }
        }

        stage('Get AKS Credentials') {
            steps {
                bat 'az aks get-credentials --resource-group %RESOURCE_GROUP% --name %AKS_CLUSTER% --overwrite-existing'
            }
        }

        stage('Deploy to AKS') {
            steps {
                bat 'kubectl apply -f deployment.yml'
            }
        }
    }
}
