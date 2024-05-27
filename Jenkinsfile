pipeline {
	
	agent any
		
	 environment {
 //    		ACR_LOGINSERVER = credentials('ACR_LOGINSERVER')
 //    		ACR_ID = credentials('ACR_ID')
	// 	    ACR_PASSWORD = credentials('ACR_PASSWORD')
		 JENKINS_USERNAME='yslee'
		 JENKINS_PASSWORD='qwer'
		    CSPM_URL = 'https://asia-1.api.cloudsploit.com'
		    AQUA_URL = 'https://api.asia-1.supply-chain.cloud.aquasec.com'
		    AQUA_KEY = credentials('AQUA_KEY')
		    AQUA_SECRET = credentials('AQUA_SECRET')
	 }
	stages{
		stage('Pipeline Enforcer Start') {
		steps{
		    sh '''
		      curl -sLo install.sh download.codesec.aquasec.com/pipeline-enforcer/install.sh
		      BINDIR="." sh install.sh
		      USERNAME=$JENKINS_USERNAME PASSWORD=$JENKINS_PASSWORD ./pipeline-enforcer ci start --aqua-key $AQUA_KEY --aqua-secret $AQUA_SECRET --cspm-url $CSPM_URL &
		    '''
		  }
		}
		
		stage ('azure-voting-app-redis - Checkout') {
			steps {
					checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/clo-dso-test/azure-voting-app-redis']]])
			}
		}
		stage ('Build, Lint, & Unit Test') {
			steps{
					//exectute build, linter, and test runner here    
					sh '''
					echo "exectute build, linter, and test runner here"
					'''
			}
		}
		stage ('Docker Build and Push to ACR'){
			steps{
					
					sh '''
					#Azure Container Registry config
					REPO_NAME="azure-voting-app-redis"
					# ACR_LOGINSERVER="lyscr222.azurecr.io"
					# IMAGE_NAME="$ACR_LOGINSERVER/$REPO_NAME:jenkins${BUILD_NUMBER}"
     					IMAGE_NAME="$REPO_NAME:jenkins${BUILD_NUMBER}"

					#Docker build and push to Azure Container Registry
					cd ./azure-vote
					docker build -t $IMAGE_NAME .
					cd ..
					
					# docker login $ACR_LOGINSERVER -u ${ACR_ID} -p ${ACR_PASSWORD}
					# docker push $IMAGE_NAME
					'''
			}
		}

		stage ('build xmrig'){
			steps {
				sh	'''
    					echo "FROM xmrig/xmrig:latest" > Dockerfile
	    				docker pull metal3d/xmrig:latest
	 				docker run --rm -d metal3d/xmrig:latest
	 				'''
			}
		}
		
	    	stage('Aqua scanner') {
		          agent {
		            docker {
		              image 'aquasec/aqua-scanner'
		            }
		          }
		          steps {
		            withCredentials([
		              string(credentialsId: 'AQUA_KEY', variable: 'AQUA_KEY'),
		              string(credentialsId: 'AQUA_SECRET', variable: 'AQUA_SECRET'),
		              string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN')
		            ]){
		              sh '''
		                export TRIVY_RUN_AS_PLUGIN=aqua
		                export AQUA_URL=https://api.asia-1.supply-chain.cloud.aquasec.com
		                export CSPM_URL=https://asia-1.api.cloudsploit.com
		                trivy fs --scanners misconfig,vuln,secret . --sast
		                # To customize which severities to scan for, add the following flag: --severity UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL
		                # To enable SAST scanning, add: --sast
		                # To enable reachability scanning, add: --reachability
		                # To enable npm/dotnet/gradle non-lock file scanning, add: --package-json / --dotnet-proj / --gradle
		                # For http/https proxy configuration add env vars: HTTP_PROXY/HTTPS_PROXY, CA-CRET (path to CA certificate)
		              '''
		            }
			}
    		}
		// stage ('Deploy to K8s'){
		// 	steps{
		// 			sh '''
		// 			# Update kubernetes deployment with new image.
  //                   WEB_IMAGE_NAME="${ACR_LOGINSERVER}/azure-vote-front:kube${BUILD_NUMBER}"
  //                   kubectl set image deployment/azure-vote-front azure-vote-front=$WEB_IMAGE_NAME
		// 			'''
		// 		}
		// }
		stage('Pipeline Enforcer End') {
		  // environment {
		  //   AQUA_KEY = credentials('AQUA_KEY')
		  //   AQUA_SECRET = credentials('AQUA_SECRET')
		  // }
		steps{
		    sh './pipeline-enforcer ci end --aqua-key $AQUA_KEY --aqua-secret $AQUA_SECRET --cspm-url $CSPM_URL'
		  }
		}
	}

	post { 
		always { 
			echo 'Build Steps Completed'
		}
	}
}
