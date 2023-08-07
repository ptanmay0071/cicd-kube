pipeline {
	agent any
	tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
	}
    environment {
        registry = "ptanmay0071/reservation"
        registryCredential = 'dockerhub'
    }

    stages{
        stage ('fetch'){
            steps{
            git branch: 'main' , url: 'https://github.com/ptanmay0071/cicd-onlinebookstore.git'
            }
        }

        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }


        stage('Building image') {
            steps{
              script {
                dockerImage = docker.build registry + ":$BUILD_NUMBER"
              }
            }
        }
        
        stage('Deploy Image') {
          steps{
            script {
              docker.withRegistry( '', registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
          }
        }

        stage('Remove Unused docker image') {
          steps{
            sh "docker rmi $registry:$BUILD_NUMBER"
          }
        }
         stage('Kubernetes Deploy') {
	  agent { label 'KOPS' }
            steps {
                    sh "helm upgrade --install --force reservation-stack helm/reservationcharts --set appimage=${registry}:${BUILD_NUMBER} --namespace prod"
            }
        }
    }
}

        