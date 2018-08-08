pipeline {
	agent { label 'master' }

	stages{
		stage("Resource Group & Credential Creation") {
			steps {
				sh 'docker run cli-image > output.json'
				script{ 
					json = readJSON file: 'output.json'
					writeFile(file: "payload.json", 
					text: """
						{
							'clientID': '${json.appId}'
							'clientSecret': '${json.password}'
							'tenantID': '${json.tenant}'
						}
					""")
				}
				withCredentials([string(credentialsId: 'root_token', variable: 'TOKEN')]) {	
					sh '''
						curl --header "X-Vault-Token $TOKEN" --request POST \
						--data @payload.json http://127.0.0.1:8200/v1/secret/${projectName}/creds
					'''
				}

				
			}
		}
	}
}