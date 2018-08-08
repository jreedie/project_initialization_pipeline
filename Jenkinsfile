pipeline {
	agent { label 'master' }

	stages{
		stage("Resource Group & Credential Creation") {
			steps {
				sh '''
					docker images
					docker run cli-image > output.txt
				'''
				
			}
		}
	}
}