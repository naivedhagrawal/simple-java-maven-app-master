@Library('Shared-Libraries') _
pipeline {
  agent none
    stages {
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
      stage ('snyk') {
        agent {
              kubernetes {
                yaml pod('snyk','snyk:maven-3-jdk-11')
              }
        }
        steps {
          checkout scm
          container('snyk'){
            sh 'snyk test'
          }
        }
      }
    }
}