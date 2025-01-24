@Library('Shared-Libraries') _
pipeline {
  agent {
    kubernetes {
      yaml pod('maven', 'maven:3.8.7')
    }
  }
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