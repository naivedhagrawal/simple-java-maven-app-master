// Jenkinsfile
@Library('k8s-shared-lib') _

pipeline {
  agent none
  environment {
        GITLEAKS_REPORT = 'gitleaks-report.json'
    }
  stages {
    stage('Gitleak Check') {
      agent {
        kubernetes {
          yaml pod('Gitleak','zricethezav/gitleaks')
          showRawYaml false
        }
      }
      steps {
        container('Gitleak') {
          sh """
              gitleaks version
              gitleaks detect --source=. --report=${env.GITLEAKS_REPORT}
              archiveArtifacts artifacts: "${env.GITLEAKS_REPORT}", allowEmptyArchive: true

          """
        }
      }
    }
    stage('Maven Build') {
      agent {
        kubernetes {
          yaml pod('maven','maven:latest')
          showRawYaml false
        }
      }
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