pipeline {
	agent { label 'master' }

	stages{
		stage("Create repo"){
			steps{
				sh '''
					mkdir ${projectName}-tmp
					cd ${projectName}-tmp

					curl -u 'repomaker:Repomaker1' https://api.github.com/user/repos \
						--data '{"name": "'${projectName}'"}'

					git init
					touch README.md
					git add README.md
					git commit -m "initial commit"
					git remote add origin https://github.com/repomaker/${projectName}.git
					git push -u https://repomaker:Repomaker1@github.com/repomaker/${projectName}.git

					cd ..
					rm -rf ${projectName}-tmp

					curl -u "repomaker:Repomaker1" -X PUT -d '' "https://api.github.com/repos/repomaker/${projectName}/collaborators/${github_username}"
				'''
			}
		}

		stage("Resource Group & Credential Creation") {
			steps {
				sh 'docker run --rm cli-image $subID $projectName > output.json'
				script{ 
					json = readJSON file: 'output.json'
					writeFile(file: "payload.json", 
					text: """{\n\t"subID": "$subID",\n\t"clientID": "${json.appId}",\n\t"clientSecret": "${json.password}",\n\t"tenantID": "${json.tenant}"\n}
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
					text: """{ "policy": "path \\"secret/$projectName/creds\\" { capabilities = [\\"read\\"] }"}""")

					sh """
						curl --header "X-Vault-Token: $TOKEN" --request PUT \
						--data @payload.json  \
						http://127.0.0.1:8200/v1/sys/policy/${projectName}-policy

						rm payload.json
					"""

					sh """
						curl --header "X-Vault-Token: $TOKEN" --request POST \
						--data '{"policies": "${projectName}-policy"}' \
						http://127.0.0.1:8200/v1/auth/approle/role/${projectName}-role 

						curl --header "X-Vault-Token: $TOKEN" \
						http://127.0.0.1:8200/v1/auth/approle/role/${projectName}-role/role-id -o roleID.json
					"""

					script{
						json = readJSON file: 'roleID.json'
						sh """
							curl --header "X-Vault-Token: $TOKEN" --request POST \
							-d '{"roleID": "${json.data.role_id}"}' http://127.0.0.1:8200/v1/secret/roles/${projectName}
						"""
					}

					writeFile(file: "payload.json", 
					text: """{ "policy": "path \\"auth/approle/role/$projectName-role/secret-id\\" { capabilities = [\\"create\\", \\"update\\"] }" }""")

					sh """
						curl --header "X-Vault-Token: $TOKEN" --request PUT \
						--data @payload.json  \
						http://127.0.0.1:8200/v1/sys/policy/${projectName}-id

						rm payload.json

						curl --header "X-Vault-Token: $TOKEN" --request POST \
						--data '{"policies": "${projectName}-id"}' \
						http://127.0.0.1:8200/v1/auth/token/create -o token.json

						cat token.json


					"""
				}

				
			}
		}


	}
}