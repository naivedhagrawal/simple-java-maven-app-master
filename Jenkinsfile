@Library('Shared-Libraries') _
pipeline {
  agent none
  environment {
        SNYK_TOKEN = credentials('SNYK_TOKEN')
        }
    stages {
      stage ('snyk') {
        agent {
              kubernetes {
                yaml snyk('maven', env.SNYK_TOKEN , false , '' , false)
              }
        }
        steps {
          checkout scm
          container('snyk'){
            script {
              snyk('maven', env.SNYK_TOKEN , false , '' , false)
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