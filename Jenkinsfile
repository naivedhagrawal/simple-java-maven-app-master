@Library('k8s-shared-lib') _

pipeline {
    agent none
    environment {
        GITLEAKS_REPORT = 'gitleaks-report.json'
        OWASP_DEP_REPORT = 'owasp-dep-report.json'
        ZAP_REPORT = 'zap-report.json'
        SEMGREP_REPORT = 'semgrep-report.json'
        TARGET_URL = 'https://google.com' // Consider making this a parameter        
        }

    stages {    
        stage('Owasp zap') {
            agent {
                kubernetes {
                    yaml pod() // Assuming this comes from your shared library and defines the ZAP pod
                    showRawYaml false
                }
            }
            steps {
                container('zap') {
                    sh 'zap-cli quick-scan $TARGET_URL'
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
                                currentBuild.result = 'FAILURE'  // Fail the build
                                // Optionally, throw an exception to stop the pipeline immediately:
                                // throw new Exception("High/Critical OWASP vulnerabilities found.")
                            } else {
                                echo "No High/Critical OWASP vulnerabilities found."
                            }
                        }
                    }
                }
            }
        }

        stage('Semgrep Scan') { // Improved Semgrep stage
            // ... (agent definition)
            steps {
                container('semgrep') {
                    sh """
                        semgrep --config=auto --output ${env.SEMGREP_REPORT} .
                    """
                    archiveArtifacts artifacts: "${env.SEMGREP_REPORT}", allowEmptyArchive: true

                    script {
                        def semgrepReport = readJSON file: "${env.SEMGREP_REPORT}"
                        // Semgrep's JSON output structure might vary. Adjust the path accordingly.
                        def criticalIssues = semgrepReport.results.findAll { it.severity == 'ERROR' || it.severity == 'WARNING' } // Check for ERROR and WARNING

                        if (criticalIssues.size() > 0) {
                            echo "Critical/Warning Semgrep Issues Found:"
                            criticalIssues.each { issue ->
                                echo "File: ${issue.path}:${issue.start.line} - ${issue.message} - Rule: ${issue.check_id}"
                            }
                            currentBuild.result = 'FAILURE'
                            // Optionally throw to stop the pipeline:
                            // throw new Exception("Critical Semgrep issues found.")
                        } else {
                            echo "No Critical/Warning Semgrep issues found."
                        }
                    }
                }
            }
        }

        stage('Maven Build') {
            // ... (agent definition)
            steps {
                container('maven') {
                    sh """
                        mvn -version
                        mvn clean package
                    """
                    // Archive your build artifacts (JAR, WAR, etc.)
                    archiveArtifacts artifacts: 'target/**' // Adjust the path as needed.
                }
            }
        }
    }
}