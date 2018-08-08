pipeline {
	agent { label 'master' }

	stages{
		stage("Resource Group & Credential Creation") {
			steps {
				
					writeFile(file: "payload.json",
					text: """{"policy": "path \"secret/$projectName/creds\" {\n\tcapabilities = ["read"]\n}"
					}""")
					sh 'cat payload.json'
				

				
			}
		}
	}
}