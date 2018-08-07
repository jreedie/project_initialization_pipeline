pipeline {
	agent any

	stages{
		stage("Resource Group & Credential Creation") {
			steps {
				withCredentials([string(credentialsId: 'root_token', variable: 'TOKEN')]) {
					azureCLI commands: [
						[exportVariablesString: '', script: 'az group create -n ${projectName} --location eastus'],
						[exportVariablesString: '/appId|clientID,/password|clientSecret,/tenant|tenantID', script: 'az ad sp create-for-rbac --scopes /subscriptions/${subID}/resourceGroups/{projectName}']
					], principalCredentialId: 'azureAdmin'
					sh '''
						set +x
						echo """
						{
							"clientID": "${clientID}"
							"clientSecret": "${clientSecret}"
							"tenantID": "${tenantID}"
						}
						""" > payload.json

						curl --header "X-Vault-Token ${TOKEN} --request POST \
						--data @payload.json" http://127.0.0.1:8200/v1/secret/${projectName}/creds
					'''
				}
			}
		}
	}
}