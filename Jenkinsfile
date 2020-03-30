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
				 withAWS(credentials: 'aws-static', region: 'us-east-2') {
					sh '''
						/home/ubuntu/bin/kubectl version --kubeconfig /var/lib/jenkins/.kube/config
					'''
				}
			}
		}

		stage('Loadbalancer traffic to blue') {
			steps {
				withAWS(credentials: 'aws-static', region: 'us-east-2') {
					sh '''
						/home/ubuntu/bin/kubectl apply -f lbtraffictoblue.yaml
					'''
				}
			}
		}

	}
}
