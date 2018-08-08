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
					sh 'cat payload.json'
				}

				
			}
		}
	}
}