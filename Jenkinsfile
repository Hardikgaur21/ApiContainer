pipeline {
    agent any

    environment {
        DOTNET_PATH = 'C:\\Program Files\\dotnet\\dotnet.exe'
        DOCKER_PATH = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker'
        TERRAFORM_PATH = 'C:\\Users\\gaurh\\Downloads\\terraform_1.11.3_windows_386\\terraform.exe'
        AZURE_CLI_PATH = 'C:\\Program Files\\Microsoft SDKs\\Azure\\CLI2\\wbin\\az'
        
        PATH = "${DOTNET_PATH};${DOCKER_PATH};${TERRAFORM_PATH};${AZURE_CLI_PATH};${PATH}"
        
        ACR_NAME = 'acrhardik01'
        AZURE_CREDENTIALS_ID = 'azure-service-principal'
        ACR_LOGIN_SERVER = "acrhardik01.azurecr.io"
        IMAGE_NAME = 'mywebapihardik'
        IMAGE_TAG = 'v1'
        RESOURCE_GROUP = 'my-rg'
        AKS_CLUSTER = 'akshardik01'
        TF_WORKING_DIR = 'terraform'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Hardikgaur21/ApiContainer.git'
            }
        }

        stage('Build .NET App') {
            steps {
                bat 'dotnet publish DotNetWebAPI\\DotNetWebAPI.csproj -c Release -o out'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %ACR_LOGIN_SERVER%/%IMAGE_NAME%:%IMAGE_TAG% -f DotNetWebAPI/Dockerfile DotNetWebAPI"
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
                bat "kubectl apply -f deployment.yml"
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
