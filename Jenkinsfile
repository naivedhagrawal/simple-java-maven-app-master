@Library('Shared-Libraries') _
pipeline {
  agent none
  environment {
        SNYK_TOKEN = credentials('SNYK_TOKEN')
        }
    stages {
      stage ('Snyk Scan') {
        agent { kubernetes { yaml snyk('maven') }
        }
        steps {
          checkout scm
          container('snyk'){
            script {
              snyk('maven')
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