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
                sh """
                    # Kill previous ZAP instances (if any)
                    pkill -f "/zap/zap.sh -daemon"
                    echo "pkill exit code: \$?"
                    sleep 5

                    mkdir -p /tmp/zap-home-${BUILD_NUMBER}

                    # Start ZAP in the foreground, then detach using nohup
                    nohup /zap/zap.sh -daemon -port 8080 -config api.disablekey=true -newsession /tmp/zap-session -home /tmp/zap-home-${BUILD_NUMBER} > /tmp/zap.out 2>&1 &  # Redirect output to a file and run in background
                    echo "ZAP started in background. Check /tmp/zap.out for logs."

                    # Give ZAP some time to start
                    sleep 30  # Adjust as needed

                    echo "Running the spider scan..."
                    /zap/zap.sh -cmd spider -url ${env.TARGET_URL} -home /tmp/zap-home-${BUILD_NUMBER}

                    echo "Running the active scan..."
                    /zap/zap.sh -cmd -config scanner.attackOnStart=false activeScan -url ${env.TARGET_URL} -format json -output ${env.ZAP_REPORT} -home /tmp/zap-home-${BUILD_NUMBER}

                    # You might want to add a step here to stop ZAP after your scans are done, if needed.
                    # For Example:
                    # pkill -f "/zap/zap.sh -daemon"
                    # echo "ZAP stopped."
                """
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