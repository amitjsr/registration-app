pipeline  {
  agent { label 'Jenkins-Agent' }
  tools {
       jdk 'Java17'
       maven 'Maven3'
  }
  environment {
            APP_NAME = "register-app-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "amit8824"
            DOCKER_PASS = 'jenkins-dockerhub'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
  }
  stages {
   stage("Cleanup workspace"){
             steps {
             cleanWs()
             }
    }
    stage("Checkout from SCM") {
               steps {
                   git branch: 'main', credentialsId: 'github', url: 'https://github.com/amitjsr/registration-app'
               }
    }
    stage("Build Application") {
        steps {
            sh "mvn clean package"
        }
    }
    stage("Test Application") {
        steps {
            sh "mvn test"
        }
    }
    stage("SonarQube Analysis"){
        steps {
           script {
              withSonarQubeEnv(credentialsId: 'Jenkins-sonarqube-token') {
              sh "mvn sonar:sonar"
                 }
           } 
       }
    }
    stage("Quality Gate"){
        steps {
            script {
                 waitForQualityGate abortPipeline: false, credentialsId: 'Jenkins-sonarqube-token'
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
    }
    stage("Trivy Scan") {
           steps {
               script {
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ashfaque9x/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
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
