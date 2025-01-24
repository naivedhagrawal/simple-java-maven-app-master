@Library('Shared-Libraries') _
pipeline {
  agent none
    stages {
      stage('Code Clone, Build, Test') {
        agent {
              kubernetes {
                yaml pod('maven', 'maven:3.8.7')
              }
        }
        steps {
          checkout scm
          container('maven') {
            sh 'mvn -B -DskipTests clean package'
            sh 'mvn test'
          }
        }
      }
      stage ('snyk') {
        agent {
              kubernetes {
                yaml snyk()
              }
        }
        steps {
          container('snyk'){
            checkout scm
            sh 'snyk code test'
          }
        }
      }
    }
}