pipeline {
    agent { label 'Jenkins-Agent' }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
	    APP_NAME = "registry-app-pipeline1"
            RELEASE = "1.0.0"
            DOCKER_USER = "samuel78996"
            DOCKER_PASS = 'dockerhub'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/slur96/register-app.git'
                }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }
       }

       stage("Test Application"){
           steps {
                 sh "mvn test"
           }
       }
	    
       stage("SonarQube Analysis"){
           steps {
	           script {
		          withSonarQubeEnv(credentialsId: 'sonarqube-token') { 
                        sh "mvn sonar:sonar"
		         }
	          }	
           }
       }
	    
	stage("Quality Gate"){
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-token'
                }	
            }
        }
	    
        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
              }
         }
	 stage("Trivy Scan") {
              steps {
               script {
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image samuel78996/registry-app-pipeline1:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
                 }
              }
           }
        }
	 stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
              }
          }
       }
    }
}
