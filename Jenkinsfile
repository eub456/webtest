pipeline {
  agent any
  import jenkins.model.*
jenkins = Jenkins.instance
  stages {
    stage('Git Progress') {
      steps {
        git branch: 'main', credentialsId: 'eub456', url: 'https://github.com/eub456/webtest.git'
      }
    }
  stage('Gradle Build & Image buil') {
      steps {
        sh 'chmod +x ./gradlew'
        sh './gradlew clean build'
        sh 'docker build -t eub456/test .'
        }
    }
    stage ('Push image') {
        steps {
                checkout scm
                  agent any  
                    'docker' docker.withRegistry('https://registry.hub.docker.com', 'test') {

                        def customImage = docker.build("eub456/test:${env.BUILD_ID}")

                            customImage.push()
                   }
        }
    }
  }
}
