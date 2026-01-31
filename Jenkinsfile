pipeline {
    agent any
    stages {
		stage("Checkout") {
            steps {
				echo "cloning the repository"
                git url: "https://github.com/shreya-yeole/Trend.git",branch: "main"
            }
        }
        stage("Build") {
            steps {
                echo "Building the image"
                sh "docker build -t trend-store-app ./dist"
                
            }
        }

        stage("Push to docker hub") {
            steps {
				echo "pushing to dockerhub"
                withCredentials([usernamePassword(credentialsId: "dockerhubCreds",
                                                 usernameVariable: "dockerHubUser",
                                                 passwordVariable: "dockerHubPass")]) {	
					sh 'echo $dockerHubPass | docker login -u $dockerHubUser --password-stdin'
					sh "docker image tag trend-store-app:latest ${env.dockerHubUser}/trend-store-app:latest"
					sh "docker push ${env.dockerHubUser}/trend-store-app:latest"
					echo "docker image pushed successfully"
                }
            }
        }
		stage('K8S Deploy') {
			steps{
				script {
					echo "deploying into kubernetes"
					withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: '031332257575']]) {
					sh "aws eks update-kubeconfig --region us-east-1 --name sky-cluster"
					sh 'kubectl apply -f deployment.yaml'
                	}
            	}
			}
    	}
	}	
    post {
        success {
            echo "✅ Build, push, and deploy completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
    }
}
