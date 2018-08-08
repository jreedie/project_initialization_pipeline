pipeline {
	agent { label 'master' }

	stages{
		stage("Resource Group & Credential Creation") {
			steps {
				sh 'docker run --rm cli-image $subID $projectName > output.json'
				script{ 
					json = readJSON file: 'output.json'
					writeFile(file: "payload.json", 
					text: """{\n\t"clientID": "${json.appId}",\n\t"clientSecret": "${json.password}",\n\t"tenantID": "${json.tenant}"\n}
					""")
				}
				sh 'cat payload.json'

				withCredentials([string(credentialsId: 'root_token', variable: 'TOKEN')]) {	
					sh """
						curl --header "X-Vault-Token: $TOKEN" --request POST \
						--data @payload.json http://127.0.0.1:8200/v1/secret/${projectName}/creds

						rm payload.json
					"""
				
					writeFile(file: "payload.json",
					text: """{"policy": "path \"secret/$projectName/creds\" {\n\tcapabilities = ["read"]\n}"\n}""")
					sh 'cat payload.json'

					sh """
						curl --header "X-Vault-Token: $TOKEN" --request PUT \
						--data @payload.json http://127.0.0.1:8200/v1/sys/${projectName}-policy

						rm payload.json
					"""
				}

				
			}
		}
	}
}