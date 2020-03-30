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
				 
					sh '''
						/home/ubuntu/bin/kubectl apply -f bluedeployment.yaml
					'''
				
			}
		}

		stage('Loadbalancer traffic to blue') {
			steps {
				
					sh '''
						/home/ubuntu/bin/kubectl apply -f lbtraffictoblue.yaml
					'''
				
			}
		}

	}
}
