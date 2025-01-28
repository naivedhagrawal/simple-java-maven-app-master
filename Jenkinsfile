// Jenkinsfile
@Library('k8s-shared-lib') _

pipeline {
  agent none
  stages {
    stage('Maven Build') {
      agent { kubernetes { yaml pod('maven','maven:latest') showRawYaml false } }
      steps {
        container('maven') {
          sh """
              mvn -version
          """
        }
      }
    }
  }
}