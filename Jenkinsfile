@Library('k8s-shared-lib') _

pipeline {
    agent none
    environment {
        GITLEAKS_REPORT = 'gitleaks-report.json'
        OWASP_DEP_REPORT = 'owasp-dep-report.json'
        ZAP_REPORT = 'zap-out.json'
        SEMGREP_REPORT = 'semgrep-report.json'
        TARGET_URL = 'http://www.itsecgames.com/'
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
                // zap-api-scan.py zap-baseline.py zap-full-scan.py zap_common.py 
                sh """
                    whoami
                    id
                    ls -ld /zap/wrk
                    touch /zap/wrk/test.json
                    touch /zap/test.json
                    touch /home/zap/test.json
                    ls -lrt /zap
                    ls -lrt /homes
                    zap-baseline.py -t ${TARGET_URL}
            
                """
                archiveArtifacts artifacts: "${env.ZAP_REPORT}", allowEmptyArchive: true
            }
            }
        }
        
        stage('Owasp Dependency Check') {
            steps {
            container('owasp') {
                withCredentials([string(credentialsId: 'NVD_API_KEY', variable: 'NVD_API_KEY')]) {
                sh """
                    dependency-check.sh --scan . --format JSON --out ${env.OWASP_DEP_REPORT} --nvdApiKey ${env.NVD_API_KEY}
                """
                archiveArtifacts artifacts: "${env.OWASP_DEP_REPORT}", allowEmptyArchive: true

                script {
                    def owaspReport = readJSON file: "${env.OWASP_DEP_REPORT}"
                    def vulnerabilities = owaspReport.report.dependencies.collectMany { it.vulnerabilities }.findAll { it.severity == 'HIGH' || it.severity == 'CRITICAL' }

                    if (vulnerabilities.size() > 0) {
                    echo "High/Critical OWASP Vulnerabilities Found:"
                    vulnerabilities.each { vuln ->
                        echo "Dependency: ${vuln.name} - ${vuln.description} - CVE: ${vuln.cve}"
                    }
                    currentBuild.result = 'FAILURE'
                    } else {
                    echo "No High/Critical OWASP vulnerabilities found."
                    }
                }
                }
            }
            }
        }

        stage('Semgrep Scan') {
            steps {
            container('semgrep') {
                sh """
                semgrep --config=auto --output ${env.SEMGREP_REPORT} .
                """
                archiveArtifacts artifacts: "${env.SEMGREP_REPORT}", allowEmptyArchive: true

                script {
                def semgrepReport = readJSON file: "${env.SEMGREP_REPORT}"
                def criticalIssues = semgrepReport.results.findAll { it.severity == 'ERROR' || it.severity == 'WARNING' }

                if (criticalIssues.size() > 0) {
                    echo "Critical/Warning Semgrep Issues Found:"
                    criticalIssues.each { issue ->
                    echo "File: ${issue.path}:${issue.start.line} - ${issue.message} - Rule: ${issue.check_id}"
                    }
                    currentBuild.result = 'FAILURE'
                } else {
                    echo "No Critical/Warning Semgrep issues found."
                }
                }
            }
            }
        }

        stage('Maven Build') {
            steps {
            container('maven') {
                sh """
                mvn -version
                mvn clean package
                """
                archiveArtifacts artifacts: 'target/**'
            }
            }
        }
        }
    }
