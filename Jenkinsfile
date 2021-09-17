pipeline {
    agent any
    stages {
        stage('Git Progress') {
            steps {
                git  branch: 'main', credentialsId: 'eub456', url: 'https://github.com/eub456/webtest.git'
            }
        }
        stage('Gradle Junit Test') {
            steps {
                sh 'chmod +x ./gradlew'
                sh "chmod +x gradlew; ./gradlew test"
            }
        }
        stage('Gradle Build ') {
            steps {
                sh 'chmod +x ./gradlew'
                sh './gradlew clean build'
            }
        }
        stage('Publish test results') {
            steps {
                junit '**/build/test-results/test/*.xml'
            }
        } 
        stage('OWASP Dependency-Check') {
            steps {
                dependencyCheck additionalArguments: '-s "./" -f "HTML" -o "./" --prettyPrint', odcInstallation: 'dependency'
            }
        }
        stage('SonarQube analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar') {
                        sh "./gradlew sonarqube \
                        -Dsonar.projectKey=sonar-test \
                        -Dsonar.host.url=http://3.36.123.126:9000/ \
                        -Dsonar.login=fb7db0007b25c4c98fa8f3e24801f3335b4211c9"
                    }
                }
            }
        }       
        stage('Anchore test') {
            steps {
                script {
                    def imageLine = 'eub456/test:latest'
                    writeFile file: 'eub456/test:latest', text: imageLine
                    anchore name: 'eub456/test:latest', engineCredentialsId: 'anchore', bailOnFail: false
                }
            }
        }
        stage('Push image') {
            steps {
                script {
                    checkout scm
                    docker.withRegistry('https://registry.hub.docker.com', 'test') {
                        def customImage = docker.build("eub456/test")
                        customImage.push("latest")
                        customImage.push("${env.BUILD_ID}")
                    }
                }
            }
        }
        stage('ArgoCD Deploy') {
            steps {
                script {
                    sshagent (credentials: ['argoCD']) {
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@54.180.113.103 argocd repo add https://github.com/eub456/webtest.git"
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@54.180.113.103 argocd app create test2 --repo https://github.com/eub456/webtest.git --sync-policy automated --path templates --dest-server https://kubernetes.default.svc --dest-namespace default"
                    }
                }
            }
        }
    }
}