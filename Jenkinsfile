@Library('Shared-Library') _
pipeline {
  agent {
    kubernetes {
      yaml pod(maven, maven:latest)
    }
  }
  stages {
    stage('Code Clone') {
        checkout scm
    }
    stage('Build') {
      steps {
        container(maven) {
          sh 'mvn -B -DskipTests clean package'
        }
      }
    }
    stage('test') {
      steps{
        container(maven) {
          sh 'mvn test'
        }
      }
    }
    stage('Deliver') {
      steps{
        container(maven) {
          sh './jenkins/scripts/deliver.sh'
        }
      }  
    }
  }
}