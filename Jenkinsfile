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
          yaml """
apiVersion: v1
kind: Pod
metadata:
  name: zap
spec:
  containers:
  - name: zap
    image: zaproxy/zap-stable
    command: ["/bin/sh", "-c"]
    args: ["tail -f /dev/null"]
"""
          showRawYaml false
        }
      }
      steps {
        container('zap') {
          sh '''
    mkdir -p /zap/workspace
    /zap/zap.sh -daemon -port 8080 -host 0.0.0.0 -config api.disablekey=true -config dirs.base=/zap/workspace &
    sleep 15  # Wait for ZAP to fully start

    # Using export to pass environment variables to the shell
    export TARGET_URL="${env.TARGET_URL}"
    export ZAP_REPORT="${env.ZAP_REPORT}"

    # Start scan
    curl "http://localhost:8080/JSON/ascan/action/scan/?url=$TARGET_URL&recurse=true&inScopeOnly=false"

    # Wait for the scan to complete
    while true; do
      STATUS=$(curl -s http://localhost:8080/JSON/ascan/view/status/ | jq -r '.status')
      if [ "$STATUS" = "100" ]; then
        break
      fi
      echo "Scanning in progress..."
      sleep 10
    done

    # Download the report
    curl "http://localhost:8080/OTHER/core/other/jsonreport/" -o "$ZAP_REPORT"

    # Shutdown ZAP
    /zap/zap.sh -cmd shutdown
'''





          archiveArtifacts artifacts: "${env.ZAP_REPORT}", allowEmptyArchive: true
        }
      }
    }

    stage('Gitleak Check') {
      agent {
        kubernetes {
          yaml """
apiVersion: v1
kind: Pod
metadata:
  name: gitleak
spec:
  containers:
  - name: gitleak
    image: zricethezav/gitleaks
    command: ["/bin/sh", "-c"]
    args: ["tail -f /dev/null"]
"""
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
          yaml """
apiVersion: v1
kind: Pod
metadata:
  name: owasp
spec:
  containers:
  - name: owasp
    image: naivedh/owasp-dep:latest
    command: ["/bin/sh", "-c"]
    args: ["tail -f /dev/null"]
"""
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

    stage('Semgrep Scan') {
      agent {
        kubernetes {
          yaml """
apiVersion: v1
kind: Pod
metadata:
  name: semgrep
spec:
  containers:
  - name: semgrep
    image: returntocorp/semgrep:latest
    command: ["/bin/sh", "-c"]
    args: ["tail -f /dev/null"]
"""
          showRawYaml false
        }
      }
      steps {
        container('semgrep') {
          sh """
            semgrep --config=auto --output ${env.SEMGREP_REPORT} .
          """
          archiveArtifacts artifacts: "${env.SEMGREP_REPORT}", allowEmptyArchive: true
        }
      }
    }

    stage('Maven Build') {
      agent {
        kubernetes {
          yaml """
apiVersion: v1
kind: Pod
metadata:
  name: maven
spec:
  containers:
  - name: maven
    image: maven:latest
    command: ["/bin/sh", "-c"]
    args: ["tail -f /dev/null"]
"""
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
