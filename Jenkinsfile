// Jenkinsfile
@Library('k8s-shared-lib') _

pipeline {
  agent none
  environment {
        GITLEAKS_REPORT = 'gitleaks-report.json'
        OWASP_DEP_REPORT = 'owasp-dep-report.json'
        ZAP_REPORT = 'zap-report.json'
        SEMGREP_REPORT = 'semgrep-report.json'
        TARGET_URL = 'https://google.com'
    }
  stages {
    stage('Owasp zap') {
      agent {
        kubernetes {
          yaml zap()
          showRawYaml false
        }
      }
      steps {
        container('zap') {
          sh """
              mkdir -p /zap/workspace
              /zap/zap.sh -daemon -port 8080 -host 0.0.0.0 -config api.disablekey=true -config dirs.base=/zap/workspace &
              sleep 15  # Wait for ZAP to fully start
              zap-cli quick-scan --self-contained --json -o ${env.ZAP_REPORT} ${env.TARGET_URL}
              /zap/zap.sh -cmd shutdown
             """
          archiveArtifacts artifacts: "${env.ZAP_REPORT}", allowEmptyArchive: true
        }
      }
    }

    
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
          // script {
          //     def owaspReport = readJSON file: "${env.OWASP_DEP_REPORT}"
          //     echo "OWASP Report: ${owaspReport}"  // Debugging line
          //     def highSeverityVulnerabilities = owaspReport.findAll { it.severity == 'High' }.size()

          //     if (highSeverityVulnerabilities > 0) {
          //       error "Build failed due to high severity vulnerabilities in dependencies."
          //     }
          //   }
          }
        }
      }
    }


    stage('Semgrep Scan') {
      agent {
        kubernetes {
          yaml pod('semgrep','returntocorp/semgrep:latest')
          showRawYaml false
        }
      }
      steps {
        container('semgrep') {
          sh """
            semgrep --config=auto --output ${env.SEMGREP_REPORT} .
          """
          archiveArtifacts artifacts: "${env.SEMGREP_REPORT}", allowEmptyArchive: true
          // script {
          //   def semgrepReport = readJSON file: "${env.SEMGREP_REPORT}"
          //   def criticalIssues = semgrepReport.findAll { it.severity == 'ERROR' }.size()
            
          //   if (criticalIssues > 0) {
          //     error "Build failed due to critical Semgrep issues."
          //   }
          // }
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