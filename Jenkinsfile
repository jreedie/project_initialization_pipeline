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

					curl -u "repomaker:Repomaker1" -X PUT -d '{"permission": "admin"}' "https://api.github.com/repos/repomaker/${projectName}/collaborators/${github_username}"
				'''
			}
		}

		stage("Create project pipeline"){
			steps{
				script{
					jobDsl scriptText: "folder('${projectName}-folder')"
					jobDsl scriptText: """
						multibranchPipelineJob('${projectName}-folder/${projectName}') {
							branchSources{
								github {
									scanCredentialsId('repo_creds')
									repoOwner('repomaker')
									repository('${projectName}')
								}
							}
						}
					"""
				}
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
					"""
				}

				
			}
		}

		stage("Initialize Folder Credentials & Add Vault Token"){
			steps{
				
				script{
					sh """
						set +x
						curl -X POST "http://jreedie:jdem99@localhost:8080/job/${projectName}-folder/credentials/store/folder/domain/_/createCredentials" \
						--data-urlencode 'json={"": "0", "credentials": {"scope": "GLOBAL", "id": "", "username": "user", "password": "", "\$class": "com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl"}}' \
						 
					"""
					json = readJSON file: 'token.json'
					injectCreds("$projectName", "${json.auth.client_token}")
				}
			}	
		}


	}
	post {
		always {
			cleanWs()
		}
	}
}