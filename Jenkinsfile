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
				withKubeConfig(caCertificate: 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01ETXpNREF6TVRZeU5Gb1hEVE13TURNeU9EQXpNVFl5TkZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTnJQCitHZENta1lQbkVYSEpMNDhWeWVJUGxJSFd4bXpUdnpGTnlLZE5tMkRJUUpUcGMvQjRUNHIwZUdaZWxxM0J1bzgKdHRtV01SUjF5TDUrVzJhMUtTalJYNFBqYWNjMXNPeUd6TUdzeW9SVjdFckt4Rkt6N2d6V01rTVB0ZHF4djJZdApoc1VRekkvYnVLZkpmT2U3MUpDdmxjTkVFeDg1Rzg2NFdTOUxEdVJxWUsyWjFiaWM5WHJQcWFyc1RUTVplZTYwCi9sbnoxUG5MdVR2Ylp6UUY4UmlGK2VaWlZMMDhyUFIxZ0dHaWtmZC9OUWVZZHVMWTNtdklYa3JtamcvMmwxTE4KMWw0Yk5ZUjZJK1drMWtGOUVlV05GTUJZeFNJYU8xeEwwaitlc0xwVTNiaUkxbHZTdCtpZzlZa2xJUjhhVkJJMQpOcUs4MzU0TTlDRVMxRXdhRnRNQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFIVzNpSW9INUpjMUZrU2k5aUFQVW4rcy9jSzgKYmZDV25nemRESFdOa3VtYlNHZ21jQWJ0d3Jia3BZWVprSFBZbkxZMGVmUGJSRll4YU1lR1lhcCtyamc2bW1ZcQplRWNZbFpSZE5qOExuSm05eHRQSlRpTEgzOEVrRG1tQThNaUlPcS85N0xvN3FVTkZabUpXRS9lYWxzb1JoS1VPCmxOc2NpcGtLeCttMG02Y2gvRDljeVphUENjYVVsNGhvVHVLcHIyWE1FcThlbXVsMndJKy8zMy9ralNyTnVCMlcKQjNiL2ErSDZ1aTBFeHljZ2NTZlQ2M2RQdDM4SkQ1L2RHQ0hMT2pkdDRjdHRGazZNMU1FSnhhY0dhZHRQSUJsNgozVGpZWTI5UHplZWpPQTFVcHZXRmpXV28vS2dFNXZiN2tTUGFRZmtQMWV3M2dnVk9XNGRRdytJL0UvOD0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=', clusterName: 'myEKSCluster', contextName: '', credentialsId: '', namespace: '', serverUrl: 'https://49611FFCD950BE63378E56B3902F791B.gr7.us-east-2.eks.amazonaws.com') {
					sh '''
						/home/ubuntu/bin/kubectl apply -f bluedeployment.yaml
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
