pipeline {
    agent {
        docker { image 'devops-agent:latest' }
    }

    environment {
        APELLIDO = "demo"
        ACR_NAME = "acrglobalcicd"
        ACR_LOGIN_SERVER = "acrglobalcicd.azurecr.io"
        IMAGE_NAME = "my-nodejs-app-apellido"
        RESOURCE_GROUP = "rg-cicd-terraform-app-araujobmw"
        AKS_NAME = "aks-dev-eastus"
    }

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World desde Jenkins!'
            }
        }

        stage('Azure Login') {
            steps {
                withCredentials([
                    string(credentialsId: 'azure-clientId',       variable: 'AZ_CLIENT_ID'),
                    string(credentialsId: 'azure-clientSecret',   variable: 'AZ_CLIENT_SECRET'),
                    string(credentialsId: 'azure-tenantId',       variable: 'AZ_TENANT_ID'),
                    string(credentialsId: 'azure-subscriptionId', variable: 'AZ_SUBSCRIPTION_ID')
                ]) {
                    sh '''
                      echo ">>> Azure login..."
                      az login --service-principal \
                        --username $AZ_CLIENT_ID \
                        --password $AZ_CLIENT_SECRET \
                        --tenant $AZ_TENANT_ID

                      az account set --subscription $AZ_SUBSCRIPTION_ID
                    '''
                }
            }
        }

        stage('Get Git Commit Short SHA') {
            steps {
                script {
                    env.IMAGE_TAG = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    echo "IMAGE_TAG generado: ${env.IMAGE_TAG}"
                }
            }
        }

        stage('Build & Push to ACR') {
            steps {
                sh '''
                  echo ">>> Login al ACR..."
                  az acr login --name $ACR_NAME

                  echo ">>> Build de imagen..."
                  docker build -t $ACR_LOGIN_SERVER/$IMAGE_NAME:$IMAGE_TAG .

                  echo ">>> Push al ACR..."
                  docker push $ACR_LOGIN_SERVER/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('AKS Credentials') {
            steps {
                sh '''
                  echo ">>> Obteniendo credenciales de AKS..."
                  az aks get-credentials \
                    --resource-group $RESOURCE_GROUP \
                    --name $AKS_NAME \
                    --overwrite-existing
                '''
            }
        }

        stage('Set Image Tag in k8s.yml') {
            steps {
                script { 
                    // Declarar más variables de entorno
                    env.API_PROVIDER_URL = "https://dev.api.com"
                    env.ENV = "dev"
                }

                sh '''
                  echo ">>> Renderizando k8s.yml..."
        
                  sed -e "s|\\${APELLIDO}|$APELLIDO|g" \
                      -e "s|\\${ENV}|$ENV|g" \
                      -e "s|\\${IMAGE_TAG}|$IMAGE_TAG|g" \
                      k8s.yml > k8s-render.yml
        
                  echo ">>> Archivo generado:"
                  cat k8s-render.yml
                '''
            }
        }

        stage('Deploy to AKS') {
          steps {
            sh '''
                az aks command invoke \
                  --resource-group rg-cicd-terraform-app-araujobmw \
                  --name aks-dev-eastus \
                  --command "kubectl apply -f k8s-render.yml" \
                  --file k8s-render.yml

            '''
          }
        }

    }
}
