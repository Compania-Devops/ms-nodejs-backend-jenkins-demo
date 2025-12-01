pipeline {
    agent {
        docker { image 'devops-agent:latest' }
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
                      az account show
                    '''
                }
            }
        }

    }
}
