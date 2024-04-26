pipeline {
	
	agent any
		
	environment {
    		ACR_LOGINSERVER = credentials('ACR_LOGINSERVER')
    		ACR_ID = credentials('ACR_ID')
		    ACR_PASSWORD = credentials('ACR_PASSWORD')
		}
	
	stages {
		
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
					ACR_LOGINSERVER="lyscr222.azurecr.io"
					IMAGE_NAME="$ACR_LOGINSERVER/$REPO_NAME:jenkins${BUILD_NUMBER}"

					#Docker build and push to Azure Container Registry
					cd ./azure-vote
					docker build -t $IMAGE_NAME .
					cd ..
					
					docker login $ACR_LOGINSERVER -u ${ACR_ID} -p ${ACR_PASSWORD}
					docker push $IMAGE_NAME
					'''
			}
	}
	
		stage ('Deploy to K8s'){
			steps{
					sh '''
					# Update kubernetes deployment with new image.
                    WEB_IMAGE_NAME="${ACR_LOGINSERVER}/azure-vote-front:kube${BUILD_NUMBER}"
                    kubectl set image deployment/azure-vote-front azure-vote-front=$WEB_IMAGE_NAME
					'''
				}
		}	
	}

	post { 
		always { 
			echo 'Build Steps Completed'
		}
	}
}
