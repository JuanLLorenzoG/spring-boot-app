def versionPom = "1.0"
pipeline {


    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Deployment
metadata:
  name: spring-boot-app
  labels:
    app: spring-boot-app
    type: back-end
spec:
  template:
    metadata:
      name: spring-boot-app
      labels:
        app: spring-boot
        type: back-end
    spec: 
      containers:
      - name: spring-boot-app
        image: "juanllorenzogomis/jenkins-nodo-java-bootcamp:1.0"
        imagePullPolicy: Always
        volumeMounts:
        - mountPath: /var/run/docker.sock
          name: docker-socket-volume
        securityContext:
          privileged: true
      volumes:
      - name: docker-socket-volume
        hostPath:
          path: /var/run/docker.sock
          type: Socket
        command:
      - sleep
      args:
      - infinity
  replicas: 1
  selector:
    matchLabels:
      app: spring-boot
      type: back-end
'''
            defaultContainer 'spring-boot-app'
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
				sh "git clone https://github.com/JuanLLorenzoG/kubernetes-helm-docker-config.git configuracion --branch test-implementation"
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
