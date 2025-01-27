@Library('Shared-Libraries') _
pipeline {
  agent none
    stages {
      stage ('snyk') {
        agent {
              kubernetes {
                yaml pod('snyk','naivedh/snyk-image:latest')
              }
        }
        steps {
          checkout scm
          container('snyk'){
            withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
              sh 'snyk auth $SNYK_TOKEN'
              sh 'snyk test'
            }
          }
        }
      }
      stage('Code Clone, Build') {
        agent {
              kubernetes {
                yaml pod('maven', 'maven:3.8.7')
              }
        }
        steps {
          checkout scm
          container('maven') {
            sh 'mvn -B -DskipTests clean package'
            
          }
        }
      }
    }
}