// Jenkinsfile
@Library('k8s-shared-lib') _

pipeline {
  agent none
  environment {
        GITLEAKS_REPORT = 'gitleaks-report.json'
        OWASP_DEP_REPORT = 'owasp-dep-report.json'
        ZAP_REPORT = 'zap-report.json'
        TARGET_URL = 'https://google.com'
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
          yaml pod('owasp','naivedh/owasp-dep:latest')
          showRawYaml false
        }
      }
      steps {
        container('owasp') {
          withCredentials([string(credentialsId: 'NVD_API_KEY', variable: 'NVD_API_KEY')]) {
          sh """
              dependency-check --scan . --format JSON --out ${env.OWASP_DEP_REPORT} --nvdApiKey ${env.NVD_API_KEY}
          """
          archiveArtifacts artifacts: "${env.OWASP_DEP_REPORT}", allowEmptyArchive: true
          }
        }
      }
    }
    stage('Owasp zap') {
      agent {
        kubernetes {
          yaml pod()
          showRawYaml false
        }
      }
      steps {
        container('zap') {
          sh """
              /zap.sh -daemon -port 8080 &
              sleep 10
              /zap.sh -cmd activeScan -url ${env.TARGET_URL} -format json -output ${env.ZAP_REPORT}
              /zap.sh -cmd shutdown
              """
          archiveArtifacts artifacts: "${env.ZAP_REPORT}", allowEmptyArchive: true
          }
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