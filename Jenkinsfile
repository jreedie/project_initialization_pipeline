pipeline {
	agent { label 'master' }

	stages{
		stage("Resource Group & Credential Creation") {
			steps {
				sh 'docker run cli-image > output.json'
				script{ 
					json = readJSON file: 'output.json'
					echo "${json.appID}"	
				}

				
			}
		}
	}
}