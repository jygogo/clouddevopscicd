pipeline {
	agent any
	stages {

		stage('Lint HTML') {
			steps {
				sh 'tidy -q -e *.html'
			}
		}

                stage('Hadolint Dockerfile') {
			steps {
				sh 'hadolint Dockerfile'
			}
		}
		
		stage('Build Docker Image') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]){
					sh '''
						docker build -t jygogo/bgimage:blue .
					'''
				}
			}
		}

		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]){
					sh '''
						docker login -u $USERNAME -p $PASSWORD
						docker push jygogo/bgimage:blue
					'''
				}
			}
		}

		stage('Deploy blue container') {
			steps {
				withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-static']]){
					sh '''
						echo /home/ubuntu/bin/kubectl version
					'''
				}
			}
		}

		stage('Loadbalancer traffic to blue') {
			steps {
				withAWS(credentials: 'aws-static', endpointUrl: 'https://49611FFCD950BE63378E56B3902F791B.gr7.us-east-2.eks.amazonaws.com', region: 'us-east-2') {
					sh '''
						/home/ubuntu/bin/kubectl apply -f lbtraffictoblue.yaml
					'''
				}
			}
		}

	}
}
