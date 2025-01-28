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
          sh """
              # Unset any previous ZAP-related environment variables to avoid conflicts
                unset ZAP_HOME
                unset HOME

                # Create a unique temp directory for ZAP home
                export ZAP_HOME=/tmp/zap-home-\$(date +%s)  # Unique directory for this run
                export HOME=\$ZAP_HOME  # Set the HOME variable to the new temp directory

                mkdir -p \$ZAP_HOME  # Ensure the directory is created

                # Set the path to the ZAP script
                export ZAP_PATH=/zap/zap.sh

                # Start ZAP with the unique home directory
                \$ZAP_PATH -daemon -port 8080 -home \$ZAP_HOME &

                sleep 10  # Wait for ZAP to initialize

                # Run the ZAP scan against the target URL
                \$ZAP_PATH -cmd activeScan -url ${env.TARGET_URL} -format json -output ${env.ZAP_REPORT}
            """
          archiveArtifacts artifacts: "${env.ZAP_REPORT}", allowEmptyArchive: true
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