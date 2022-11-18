def versionPom = "1.0"
pipeline {


    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: juanllorenzogomis/jenkins-nodo-java-bootcamp:1.0
    command:
    - sleep
    args:
    - infinity
'''
            defaultContainer 'shell'
        }
    }


	environment {
		DOCKERHUB_CREDENTIALS=credentials("jenkins_dockerhub")
		DOCKER_IMAGE_NAME="juanllorenzogomis/spring-boot-app"
	}

	stages {
		stage("Build"){
			steps {
				sh "mvn clean package -DskipTests"
			}
		}

		stage("Build image and push to Docker Hub") {
			steps{
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
				sh "docker build -t $DOCKER_IMAGE_NAME:${versionPom} ."
				sh "docker push $DOCKER_IMAGE_NAME:${versionPom}"
				sh "docker build -t $DOCKER_IMAGE_NAME:latest ."
				sh "docker push $DOCKER_IMAGE_NAME:latest"
			}
		}

		stage("Deploy to K8s") {
			steps{
				sh "rm -rf configuracion"
				sh "git clone https://github.com/JuanLLorenzoG/kubernetes-helm-docker-config.git configuracion --branch demo-java"
				sh "kubectl apply -f configuracion/kubernetes-deployments/spring-boot-app/deployment.yaml --kubeconfig=configuracion/kubernetes-config/config"
			}
		}
	}

	post {
		always {
			sh "docker logout"
		}
	}

}
