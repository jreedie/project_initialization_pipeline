pipeline {
	agent { label 'master' }

	stages{
		stage("Resource Group & Credential Creation") {
			steps {
				
					writeFile(file: "payload.json",
					text: """{"policy": "path \"secret/$projectName/creds\"" {
						capabilities = ["read"]
					}
					}""")
					sh 'cat payload.json'
				

				
			}
		}
	}
}