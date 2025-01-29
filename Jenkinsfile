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
        ZAP_API_URL = "http://localhost:8080"
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
    stage('Owasp zap') {
      agent {
        kubernetes {
          yaml zap()
          showRawYaml false
        }
      }
      steps {
        container('zap') {
          script {
        // Use withCredentials to retrieve the ZAP API key from Jenkins Credentials
        withCredentials([string(credentialsId: 'ZAP_API_KEY', variable: 'ZAP_API_KEY')]) {
          
          // Perform Active Scan via the API
          def activeScanUrl = "${env.ZAP_API_URL}/JSON/ascan/action/scan"
          def activeScanParams = [
            url: env.TARGET_URL,
            apikey: env.ZAP_API_KEY
          ]
          
          // Trigger the active scan using ZAP API
          def activeScanResponse = httpRequest(
            url: activeScanUrl,
            httpMode: 'POST',
            query: activeScanParams,
            validResponseCodes: '200'
          )
          echo "Active scan triggered: ${activeScanResponse}"
          
          // Wait for the scan to complete (you may want to adjust the sleep time)
          sleep(time: 60, unit: 'SECONDS')  // Adjust sleep time based on your scan duration
          
          // Fetch the scan result in JSON format
          def scanResultsUrl = "${env.ZAP_API_URL}/JSON/core/view/alerts"
          def scanResults = httpRequest(
            url: scanResultsUrl,
            httpMode: 'GET',
            query: [apikey: env.ZAP_API_KEY, baseurl: env.TARGET_URL],
            validResponseCodes: '200'
          )
          
          // Save the scan result to a file
          writeFile file: "${env.ZAP_REPORT}", text: scanResults
          
          // Archive the ZAP report in JSON format
              archiveArtifacts artifacts: "${env.ZAP_REPORT}", allowEmptyArchive: true
            }
          }
        }
      }
    }
    // stage('Maven Build') {
    //   agent {
    //     kubernetes {
    //       yaml pod('maven','maven:latest')
    //       showRawYaml false
    //     }
    //   }
    //   steps {
    //     container('maven') {
    //       sh """
    //           mvn -version
    //           mvn clean package
    //       """
    //     }
    //   }
    // }
  }
}