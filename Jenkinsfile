@Library('k8s-shared-lib') _

pipeline {
    agent none
    environment {
        TARGET_URL = 'https://juice-shop.herokuapp.com/'
    }

    stages {    

        stage('Set Date to IST') {
            steps {
                script {
                    // Get the current date in IST
                    def sdf = new java.text.SimpleDateFormat("yyyy-MM-dd_HH-mm-ss")
                    sdf.setTimeZone(java.util.TimeZone.getTimeZone("Asia/Kolkata"))
                    def IST_DATE = sdf.format(new Date())
                    echo "Timestamp in IST: ${IST_DATE}"

                    // Setting the IST_DATE in environment variable
                    env.IST_DATE = IST_DATE

                    // Dynamically setting the report filenames
                    env.GITLEAKS_REPORT = "gitleaks-report-${env.JOB_NAME}-${env.BUILD_NUMBER}-${env.IST_DATE}.json"
                    env.OWASP_DEP_REPORT = "owasp-dep-report-${env.JOB_NAME}-${env.BUILD_NUMBER}-${env.IST_DATE}.json"
                    env.ZAP_REPORT = "zap-out-${env.JOB_NAME}-${env.BUILD_NUMBER}-${env.IST_DATE}.json"
                    env.SEMGREP_REPORT = "semgrep-report-${env.JOB_NAME}-${env.BUILD_NUMBER}-${env.IST_DATE}.json"
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
                    archiveArtifacts artifacts: "${env.GITLEAKS_REPORT}"
                }
            }
        }
        stage('Owasp Dependency Check') {
            agent {
                kubernetes {
                    yaml pod('owasp','naivedh/owasp-dependency:latest')
                    showRawYaml false
                }
            }
            steps {
            container('owasp') {
                withCredentials([string(credentialsId: 'NVD_API_KEY', variable: 'NVD_API_KEY')]) {
                sh """
                    dependency-check --scan . --format JSON --out ${env.OWASP_DEP_REPORT} --nvdApiKey ${env.NVD_API_KEY}
                """
                archiveArtifacts artifacts: "${env.OWASP_DEP_REPORT}"

                // script {
                //     def owaspReport = readJSON file: "${env.OWASP_DEP_REPORT}"
                //     def vulnerabilities = owaspReport.report.dependencies.collectMany { it.vulnerabilities }.findAll { it.severity == 'HIGH' || it.severity == 'CRITICAL' }

                //     if (vulnerabilities.size() > 0) {
                //     echo "High/Critical OWASP Vulnerabilities Found:"
                //     vulnerabilities.each { vuln ->
                //         echo "Dependency: ${vuln.name} - ${vuln.description} - CVE: ${vuln.cve}"
                //     }
                //     currentBuild.result = 'FAILURE'
                //     } else {
                //     echo "No High/Critical OWASP vulnerabilities found."
                //     }
                // }
                }
            }
            }
        }

        stage('Semgrep Scan') {
            agent {
                kubernetes {
                    yaml pod('semgrep','returntocorp/semgrep')
                    showRawYaml false
                }
            }
            steps {
            container('semgrep') {
                sh """
                semgrep --config=auto --output ${env.SEMGREP_REPORT} .
                """
                archiveArtifacts artifacts: "${env.SEMGREP_REPORT}"

                // script {
                // def semgrepReport = readJSON file: "${env.SEMGREP_REPORT}"
                // def criticalIssues = semgrepReport.results.findAll { it.severity == 'ERROR' || it.severity == 'WARNING' }

                // if (criticalIssues.size() > 0) {
                //     echo "Critical/Warning Semgrep Issues Found:"
                //     criticalIssues.each { issue ->
                //     echo "File: ${issue.path}:${issue.start.line} - ${issue.message} - Rule: ${issue.check_id}"
                //     }
                //     currentBuild.result = 'FAILURE'
                // } else {
                //     echo "No Critical/Warning Semgrep issues found."
                // }
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
                archiveArtifacts artifacts: 'target/**'
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
                // zap-api-scan.py zap-baseline.py zap-full-scan.py zap_common.py 
                sh """
                    zap-baseline.py -t $TARGET_URL -J $ZAP_REPORT -l WARN -I
                    mv /zap/wrk/${ZAP_REPORT} .
                """
                archiveArtifacts artifacts: "${env.ZAP_REPORT}"
            }
            }
        }
        }
    }
