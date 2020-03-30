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
						sudo docker build -t jygogo/bgimage:blue .
					'''
				}
			}
		}

		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]){
					sh '''
						sudo docker login -u $USERNAME -p $PASSWORD
						sudo docker push jygogo/bgimage:blue
					'''
				}
			}
		}

		stage('Set kubectl context') {
			steps {
				withAWS(region:'us-east-2', credentials:'aws-static') {
					sh '''
						kubectl config use-context arn:aws:eks:us-east-2:966330518435:cluster/myEKSCluster
					'''
				}
			}
		}

		stage('Deploy blue container') {
			steps {
				withAWS(region:'us-east-2', credentials:'aws-static') {
					sh '''
						kubectl apply -f bluedeployment.yaml
					'''
				}
			}
		}

		stage('Loadbalancer traffic to blue') {
			steps {
				withAWS(region:'us-east-2', credentials:'aws-static') {
					sh '''
						kubectl apply -f lbtraffictoblue.yaml
					'''
				}
			}
		}

	}
}
