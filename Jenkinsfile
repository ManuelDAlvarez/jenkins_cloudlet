pipeline {
	agent any
	parameters {
        choice(name: 'NETWORK', choices: ['staging', 'production'], description: 'The network to activate the configuration.')
    	string(name: 'CONFIGNAME', defaultValue: "${env.BUILD_ID}", description: 'The domain you are adding to the configurations') 
    }
    environment {
    	PROJ = "/bin:/usr/local/bin:/usr/bin"
	}
	stages {
		
		stage('Messages') {
			steps {
				echo "You say ${NETWORK} for ${CONFIGNAME}"
				//slackSend(botUser: true, message: "${env.JOB_NAME} will create ${CONFIGNAME} add it to the DPP and push it to ${NETWORK} for ", color: '#1E90FF')
				//slackSend(botUser: true, message: "${env.JOB_NAME} will create ${CONFIGNAME}, add it to the DPP, and push it to ${NETWORK} ", color: '#777555')
			}
		}
		stage('cloneconfig') {
			steps {
				withEnv(["PATH+EXTRA=$PROJ"]) {
					sh "akamai property create ${CONFIGNAME} --clone manuel.demo.com --hostnames ${CONFIGNAME} --edgehostname manuel.origin.edgekey.net"
				}
			}
		}
		stage('updateJSON') {
			steps {
				withEnv(["PATH+EXTRA=$PROJ"]) {
					sh "akamai property retrieve ${CONFIGNAME} --file metadata.json"
					archiveArtifacts "metadata.json"
					sh "sed 's/371349/378312/g' metadata.json > cpreplace.json"
					archiveArtifacts "cpreplace.json"
					sh "sed 's/\"customStatKey\": \"default\"//g' cpreplace.json > statkey.json"
					archiveArtifacts "statkey.json"
					sh "sed 's/toHost\": \"www.kloth.net\",/toHost\": \"www.kloth.net\"/g' statkey.json > 2metadata.json"
					archiveArtifacts "2metadata.json"
					//sh "cat 2metadata.json | grep 'toHost' "
				}
			}
		}
		stage('updateConfiguration') {
			steps {
				withEnv(["PATH+EXTRA=$PROJ"]) {
					sh "akamai property update ${CONFIGNAME} --file 2metadata.json"
				}
			}
		}
		stage('activatedelivery') {
			steps {
				withEnv(["PATH+EXTRA=$PROJ"]) {
					sh "akamai property activate ${CONFIGNAME} --network ${NETWORK}"
				}
			}
		}
		stage('CreateNewVersion') {
			steps {
				withEnv(["PATH+EXTRA=$PROJ"]) {
					sh "akamai appsec --section default clone --config=40539 --version=STAGING"
					//echo "yes activatedelivery"
				}
			}
		}
		stage('AddHostname') {
			steps {
				withEnv(["PATH+EXTRA=$PROJ"]) {
					sh "akamai appsec --section default add-hostname --config=40539 ${CONFIGNAME}"
				}
			}
		}
		stage('ClonePolicy') {
			steps {
				withEnv(["PATH+EXTRA=$PROJ"]) {
					script {
                 	   def policyId = sh(script: "akamai appsec --section default clone-policy ODPP_78900 --config=40539  --prefix=7${env.BUILD_ID}  --json | jq '.policyId'", returnStdout: true).trim()
                 	   println("policyId = ${policyId}")
                 	   sh(script: "akamai appsec --section default create-match-target --config=40539 --hostnames=${CONFIGNAME} --paths='/*' --policy=${policyId} ", returnStdout: true).trim()
                 	   
                	}
				}
			}
		}
		stage('ActivateConfig') {
			steps {
				withEnv(["PATH+EXTRA=$PROJ"]) {
					sh "akamai appsec activate --section default --config=40539 --network=${NETWORK} --notes='automated' --notify=maalvare@akamai.com"
				}
			}
		}

		stage('Cloudlet') {
			steps {
				withEnv(["PATH+EXTRA=$PROJ"]) {
					//sh "http --auth-type edgegrid -a default: GET :/cloudlets/api/v2/policies/17562/activations | jq '.[] | select(.network == \"prod\")' | jq '.policyInfo | select (.status == \"active\")' | jq .version"
					script {
                 	   def clodV = sh(script: "http --auth-type edgegrid -a default: GET :/cloudlets/api/v2/policies/17562/activations | jq '.[] | select(.network == \"prod\")' | jq '.policyInfo | select (.status == \"active\")' | jq .version | uniq", returnStdout: true).trim()
                 	   println("clodV = ${clodV}")
                 	   sh(script: "echo '{\"additionalPropertyNames\":[\"${CONFIGNAME}\"],\"network\":\"staging\",\"version\":\"${clodV}\"}' | http -v --auth-type edgegrid -a default: POST :/cloudlets/api/v2/policies/17562/versions/${clodV}/activations", returnStdout: true).trim()                 	   
                	}
				}
			}
		}

	}
	post {
    	success {
            echo "success"
            //slackSend(botUser: true, message: "${env.JOB_NAME} - ${CONFIGNAME} deployed and secured.", color: '#008000')
        }
        failure {
        	echo "failure"
            //slackSend(botUser: true, message: "${env.JOB_NAME} - Onboarding of ${CONFIGNAME} failed!", color: '#FF0000')
        }
    }
}
