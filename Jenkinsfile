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
          script {
                    def zapPodName = 'zap' // Name of the ZAP pod
                    def zapService = 'zap-service' // Name of the ZAP service (explained below)
                    def targetUrl = 'http://<your-application-service>:<your-application-port>' // URL of your application in Kubernetes

                    // 1. Create a Service for the ZAP pod (Important for accessibility within the cluster)
                    sh "kubectl expose pod ${zapPodName} --port=8080 --target-port=8080 --name=${zapService}"

                    // 2. Wait for ZAP to start (Check the logs for confirmation) - Improve with proper readiness probe
                    sleep(time: 30, unit: 'SECONDS') // Adjust as needed

                    // 3. Run the ZAP scan using the ZAP API - Example using `curl`
                    sh """
                        curl -X POST "http://${zapService}:8080/JSON/core/action/zap/newScan" -H "Content-Type: application/json" -d '{"url": "${targetUrl}", "recurse": true}'
                    """

                    // 4. Get the scan ID
                    def scanId = sh(returnStdout: true, script: """
                        curl -s "http://${zapService}:8080/JSON/core/view/lastScan" | jq -r '.scan.id'
                    """).trim()

                    // 5. Poll for scan completion (Improve with proper status checks)
                    while (true) {
                        def scanStatus = sh(returnStdout: true, script: """
                            curl -s "http://${zapService}:8080/JSON/core/view/scanProgress?scanId=${scanId}" | jq -r '.scan.progress'
                        """).trim()
                        echo "Scan Progress: ${scanStatus}%"
                        if (scanStatus == '100') {
                            break
                        }
                        sleep(time: 10, unit: 'SECONDS')
                    }

                    // 6. Generate and publish the ZAP report
                    sh """
                        curl -X GET "http://${zapService}:8080/JSON/core/other/generateReport" -H "Content-Type: application/json" -d '{"type":"html","fileName":"zap_report.html"}' > zap_report.html
                    """
                    publishHTML([path: 'zap_report.html', description: 'ZAP Scan Report'])

                    // 7. Delete the ZAP service (Cleanup)
                    sh "kubectl delete service ${zapService}"
                }
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