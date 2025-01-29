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
                    yaml zap() // Assuming this comes from your shared library and defines the ZAP pod
                    showRawYaml false
                }
            }
            steps {
                container('zap') {
                    script {
                        def targetUrl = env.TARGET_URL // Assuming TARGET_URL is defined as a Jenkins environment variable

                        // 1. Start ZAP daemon (if not already running - best practice is to manage this separately)
                        // This is generally handled outside of the pipeline in a Docker Compose setup or similar
                        // sh 'docker run -d -p 8080:8080 -v zap_data:/zap/.ZAP_HOME zap-image zap -daemon -host 0.0.0.0'

                        // 2. Get the ZAP API key (replace with your actual key or retrieve it if needed)
                        def zapApiKey = env.ZAP_API_KEY ?: sh(returnStdout: true, script: 'cat /zap/.ZAP_HOME/config/apikey').trim()  // Retrieve if not in env

                        // 3. Run the ZAP scan using the API (recommended approach)
                        def zapApiUrl = "http://localhost:8080/JSON/core/action/scan" // Default ZAP API URL (adjust if needed)

                        def response = httpRequest(
                            url: zapApiUrl,
                            httpMode: 'POST',
                            requestBody: "apikey=${zapApiKey}&url=${targetUrl}&recurse=true", // Add recurse=true for recursive scan
                            contentType: 'application/x-www-form-urlencoded', // Important for API calls
                            timeout: 60000 // Timeout in milliseconds (adjust as needed)
                        )

                        // 4. Process the API response (e.g., check for errors, get scan ID)
                        def responseJson = readJSON text: response.content

                        if (responseJson.core.scan != null && responseJson.core.scan.id != null) {
                            def scanId = responseJson.core.scan.id
                            echo "ZAP scan started with ID: ${scanId}"

                            // 5. (Optional) Poll for scan status (if needed)
                            // You can use the scan ID to periodically check the scan status using the API:
                            // http://localhost:8080/JSON/core/view/progress?apikey=<API_KEY>&scanId=<SCAN_ID>
                            // ... (Add polling logic here)

                        } else {
                            echo "Error starting ZAP scan: ${response.content}"
                            currentBuild.result = 'UNSTABLE' // Or 'FAILURE' depending on your needs
                        }

                        // 6. (Optional) Generate a report (after the scan completes)
                        // You can use the ZAP API to generate different types of reports.
                        // Example:
                        // sh "curl -X GET 'http://localhost:8080/JSON/reports/action/generateReport?apikey=${zapApiKey}&type=html&fileName=/zap/report.html'"
                        // Or using httpRequest:
                        // ...

                    }
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