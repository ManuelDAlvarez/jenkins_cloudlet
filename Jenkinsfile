pipeline {
	agent any
	parameters {
        choice(name: 'NETWORK', choices: ['staging', 'production'], description: 'The network to activate the configuration.')
    	string(name: 'CONFIGNAME', defaultValue: "${env.BUILD_ID}", description: 'The domain you are adding to the configurations') 
    	//string(name: 'CONFIGNAME', defaultValue: "manueltest33.edgesuite.net", description: 'The new domain name') 
    }
    environment {
    	PROJ = "/bin:/usr/local/bin:/usr/bin"
	}
	stages {
		
		stage('Cloudlet') {
			steps {
				withEnv(["PATH+EXTRA=$PROJ"]) {
					sh "http --auth-type edgegrid -a default: :/cloudlets/api/v2/policies/17562/activations | jq '.[] | select(.network == \"prod\")'"
					sh "http --auth-type edgegrid -a default: :/cloudlets/api/v2/policies/17562/activations | jq '.[] | select(.network == \"prod\")' | jq '.policyInfo | select (.status == \"active\")'"
					sh "http --auth-type edgegrid -a default: :/cloudlets/api/v2/policies/17562/activations | jq '.[] | select(.network == \"prod\")' | jq '.policyInfo | select (.status == \"active\")' | jq .version"
					sh "http --auth-type edgegrid -a default: :/cloudlets/api/v2/policies/17562/activations | jq '.[] | select(.network == \"prod\")' | jq '.policyInfo | select (.status == \"active\")' | jq .version | uniq"
					script {
                 	   def clodV = sh(script: "http --auth-type edgegrid -a default: :/cloudlets/api/v2/policies/17562/activations | jq '.[] | select(.network == \"prod\")' | jq '.policyInfo | select (.status == \"active\")' | jq .version | uniq", returnStdout: true).trim()
                 	   println("clodV = ${clodV}")
                 	//   sh(script: "echo '{\"additionalPropertyNames\":[\"${clodV}\"],\"network\":\"staging\",\"version\":\"4\"}' | http -v --auth-type edgegrid -a default: POST :/cloudlets/api/v2/policies/17562/versions/${clodV}/activations", returnStdout: true).trim()                 	   
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
