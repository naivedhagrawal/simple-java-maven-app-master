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
        ZAP_SERVICE_NAME = 'zap-service' // Name of your ZAP service
        ZAP_NAMESPACE = 'devops-tools' // Namespace of your ZAP service
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
          script {
            withCredentials([string(credentialsId: 'ZAP_API_KEY', variable: 'ZAP_API_KEY')]) {
              // Construct ZAP API URL dynamically using Kubernetes service discovery
              def zapApiUrl = "http://${env.ZAP_SERVICE_NAME}.${env.ZAP_NAMESPACE}:9090" // Correct port

                            // Function to check ZAP scan status
                            def checkZapScanStatus(String zapApiUrl, String apiKey, String scanId) {
                              def statusUrl = "${zapApiUrl}/JSON/ascan/view/status"
                              def statusParams = [apikey: apiKey, scanId: scanId]
              
                              try {
                                    def statusResponse = httpRequest(
                                        url: statusUrl,
                                        httpMode: 'GET',
                                        query: statusParams,
                                        validResponseCodes: '200'
                                    )

                                    def statusJson = readJSON text: statusResponse.content
                                    def scanStatus = statusJson.ascan.status
                                    echo "ZAP Scan Status: ${scanStatus}"
                                    return scanStatus
                                } catch (Exception e) {
                                    echo "Error checking ZAP status: ${e.message}"
                                    return "-1" // Indicate an error
                                }
                            }

                            // Trigger Active Scan
                            def activeScanUrl = "${zapApiUrl}/JSON/ascan/action/scan"
                            def activeScanParams = [url: env.TARGET_URL, apikey: env.ZAP_API_KEY]

                            try {
                                def activeScanResponse = httpRequest(
                                    url: activeScanUrl,
                                    httpMode: 'POST',
                                    query: activeScanParams,
                                    validResponseCodes: '200'
                                )
                                def scanId = readJSON(text: activeScanResponse.content).scanid // Extract scan ID
                                echo "Active scan triggered. Scan ID: ${scanId}"

                                // Poll for scan completion
                                while (checkZapScanStatus(zapApiUrl, env.ZAP_API_KEY, scanId) != "100") {
                                    sleep(time: 10, unit: 'SECONDS')
                                }
                                echo "ZAP Scan Complete!"

                                // Fetch and save scan results
                                def scanResultsUrl = "${zapApiUrl}/JSON/core/view/alerts"
                                def scanResultsParams = [apikey: env.ZAP_API_KEY, baseurl: env.TARGET_URL]
                                def scanResults = httpRequest(
                                    url: scanResultsUrl,
                                    httpMode: 'GET',
                                    query: scanResultsParams,
                                    validResponseCodes: '200'
                                )
                                writeFile file: "${env.ZAP_REPORT}", text: scanResults.content
                                archiveArtifacts artifacts: "${env.ZAP_REPORT}", allowEmptyArchive: true

                        } catch (Exception e) {
                            echo "Error during ZAP scan: ${e.message}"
                            currentBuild.result = 'FAILURE' // Fail the build
                            throw e // Stop the pipeline
                        }
                    }
                }
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