@Library('Shared-Libraries') _
pipeline {
  //Agent setup
  agent {
    kubernetes {
      yaml pod('maven', 'maven:3.8.7')
    }
  }
  // Stages Steps and Container
  stages {
    stage('Code Clone') {
      steps {
        checkout scm
      }
    }
    stage('Build') {
      steps {
        container('maven') {
          sh 'mvn -B -DskipTests clean package'
        }
      }
    }
    stage('test') {
      steps{
        container('maven') {
          sh 'mvn test'
        }
      }
    }
  }
}