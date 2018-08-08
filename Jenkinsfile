pipeline {
	agent { label 'master' }

	stages{
		stage("Resource Group & Credential Creation") {
			steps {
				sh 'docker run cli-image > output.json'
				script{ 
					json = readJSON file: 'output.json'
					sh '''
						echo """
							"clientID": "${json.appId}"
							"clientSecret": "${json.password}"
							"tenantID": "${json.tenant}"
						""" > payload.json
					'''
					sh 'cat payload.json'
				}

				
			}
		}
	}
}