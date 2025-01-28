// Jenkinsfile
@Library('k8s-shared-lib') _

pipeline {
  agent none
  environment {
        GITLEAKS_REPORT = 'gitleaks-report.json'
        OWASP_DEP_REPORT = 'owasp-dep-report.json'
    }
  stages {
    stage('Gitleak Check') {
      agent {
        kubernetes {
          yaml pod('gitleak','zricethezav/gitleaks')
          showRawYaml false
        }
      }
      steps {
        container('gitleak') {
          sh """
              gitleaks version
              gitleaks detect --source=. --report-path=${env.GITLEAKS_REPORT}
          """
          archiveArtifacts artifacts: "${env.GITLEAKS_REPORT}", allowEmptyArchive: true
        }
      }
    }
    stage('Owasp Dependency Check') {
      agent {
        kubernetes {
          yaml pod('owasp','owasp/dependency-check-action:latest')
          showRawYaml false
        }
      }
      steps {
        container('owasp') {
          sh """
            ls -al /usr/local/bin/
            dependency-check.sh --scan . --format JSON --out ${env.OWASP_DEP_REPORT}
          """
          archiveArtifacts artifacts: "${env.OWASP_DEP_REPORT}", allowEmptyArchive: true
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
              mvn clean package
          """
        }
      }
    }
  }
}