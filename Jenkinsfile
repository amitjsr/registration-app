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
            DOCKER_PASS = 'Amit@8824'
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
    
  }
}
