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
          sh """
                # Create workspace directory
                mkdir -p /zap/workspace
                
                # Install jq to parse JSON
                apt-get update && apt-get install -y jq

                # Start ZAP in daemon mode
                /zap/zap.sh -daemon -port 8080 -host 0.0.0.0 -config api.disablekey=true -config dirs.base=/zap/workspace &

                # Wait for ZAP to fully start
                echo "Waiting for ZAP to start..."
                sleep 30  # Increased wait time

                # Add URL to ZAP context
                echo "Adding URL to ZAP context: ${env.TARGET_URL}"
                curl "http://localhost:8080/JSON/core/action/accessUrl/?url=${env.TARGET_URL}"

                # Start the active scan
                echo "Attempting to scan URL: ${env.TARGET_URL}"
                curl "http://localhost:8080/JSON/ascan/action/scan/?url=${env.TARGET_URL}&recurse=true&inScopeOnly=false"

                # Wait for the scan to complete
                echo "Waiting for scan to complete..."
                while [ "$(curl -s http://localhost:8080/JSON/ascan/view/status/ | jq -r '.status')" != "100" ]; do
                    echo "Scanning in progress..."
                    sleep 10  # Check every 10 seconds
                done

                # Download the scan report
                echo "Scan complete, downloading the report..."
                curl "http://localhost:8080/OTHER/core/other/jsonreport/" -o ${env.ZAP_REPORT}

                # Shutdown ZAP
                /zap/zap.sh -cmd shutdown
            """


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
