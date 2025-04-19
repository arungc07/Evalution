Questions:
	1.	Fork the project http://github.com/adikarthick/project
	2.	follow the instructions provided and setup e2e jenkinspipeline and run the application
	3.	push the built images to docker hub
	4.	run the containers with new images
	5.	application should be accessible on port 9095 over a loadbalancer

Step 1: 	Fork the project http://github.com/adikarthik/project:

		We will fork the repo to our repo from http://github.com/adikarthik/project
 

Step 2.	Follow the instructions provided and setup e2e Jenkins pipeline and run the application

pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'arungc07' // <-- Replace this!
        BACKEND_IMAGE = "${DOCKER_HUB_REPO}/project-backend:latest"
        FRONTEND_IMAGE = "${DOCKER_HUB_REPO}/project-frontend:latest"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch:'main', url: 'https://github.com/arungc07/Project.git'
            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'mvn clean install'
                    sh 'docker build -t $BACKEND_IMAGE .'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh 'mvn clean install'
                    sh 'docker build -t $FRONTEND_IMAGE .'
                }
            }
        }

    stage('Docker Login and Push Images') {
    steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            sh '''
                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                docker push $BACKEND_IMAGE
                docker push $FRONTEND_IMAGE
            '''
        }
    }
        }

        stage('Run Containers') {
            steps {
                sh 'docker stop backend || true && docker rm backend || true'
                sh 'docker stop frontend || true && docker rm frontend || true'

                sh 'docker run -d -p 8085:8080 --name backend $BACKEND_IMAGE'
                sh 'export BACKEND=http://135.235.156.235:8085 && docker run -d -p 8084:8080 --name frontend $FRONTEND_IMAGE'
            }
        }
    }

    post {
        success {
            echo "✅ Application deployed:"
            echo "Frontend: http://135.235.156.235:8084"
            echo "Backend Docs: http://135.235.156.235:8085/docs"
        }
        failure {
            echo "❌ Build or deployment failed!"
        }
    }
}
=================================================================================

3. Push the built images to docker hub
4. Will tag the image and will push to docker hub
   
docker tag backend:latest arungc0/backend:latest
docker tag frontend:latest arungc07/frontend:latest
docker Push backend:latest arungc07/backend:latest
docker Push frontend:latest arungc07/:frontend:latest
================================================

6. Run the containers with new images

docker run -d --name backend -p 8088:8080 frontend:latest

docker run -d --name frontend -e BACKEND=http://135.235.156.235/:9095/api -p 8089:8080 backend:latest


1.	Fork the project http://github.com/adikarthick/project   --- Completed
2.	follow the instructions provided and setup e2e jenkinspipeline and run the application – Completed 
3.	push the built images to docker hub – completed 
4.	run the containers with new images -- completed
5.	application should be accessible on port 9095 over a loadbalancer
