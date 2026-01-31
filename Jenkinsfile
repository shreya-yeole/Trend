pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo "Building the image"
                sh 'docker build -t trend-store-app ./dist'
                
            }
        }

        stage('Push to docker hub') {
            steps {
				echo "pushing to dockerhub"
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                 usernameVariable: 'DOCKER_USER}//tredstore',
                                                 passwordVariable: 'DOCKER_PASS')]) {						 
                    sh "docker login -u ${env.DOCKER_USER} -p ${env.DOCKER_PASS}"
					sh "docker push ${env.DOCKER_USER}/tredstore:latest"
					echo "docker image pushed successfully"
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
