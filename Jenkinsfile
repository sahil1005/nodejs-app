pipeline {
    agent any
	
	environment {
		PROJECT_ID = 'ppedetonline'
                CLUSTER_NAME = 'nodejs-cluster'
                LOCATION = 'asia-south1-a'
                CREDENTIALS_ID = 'kubernetes'
						
	}
	
    stages {
	    stage('Scm Checkout') {
		    steps {
			    checkout scm
		    }
	    }
	    
	    stage('Build Docker images') {
		    steps {
				sh 'whoami'
			    script {
					myimage = docker.build("asia.gcr.io/ppedetonline/nodejs-test:${env.BUILD_ID}", "./app") 
					} 
				}
	    }

		stage("Pushing image to gcr.io") {
		    steps {
			    script {
				   withCredentials([file(credentialsId: 'gcr', variable: 'GC_KEY')]){
              sh "cat '$GC_KEY' | docker login -u _json_key --password-stdin https://asia.gcr.io"
              sh "gcloud auth activate-service-account --key-file='$GC_KEY'"
              sh "gcloud auth configure-docker"
              GLOUD_AUTH = sh (
                    script: 'gcloud auth print-access-token',
                    returnStdout: true
                ).trim()
              echo "Pushing image To GCR"
              sh "docker push asia.gcr.io/ppedetonline/nodejs-test:${env.BUILD_ID}"
          			}  
			    }
		    }
	    }

	    
	    stage('Deploy to K8s') {
		    steps{
			    echo "Deployment started ..."
			    sh 'ls -ltr'
			    sh 'pwd'
			    sh "sed -i 's/tagversion/${env.BUILD_ID}/g' deployment.yaml"
			    echo "Start deployment of deployment.yaml"
			    step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
				echo "Start deployment of service.yaml"
				step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'service.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
			    echo "Deployment Finished ..."
		    }
	    }
    }
}