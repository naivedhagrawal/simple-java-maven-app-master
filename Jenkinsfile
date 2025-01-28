@Library('k8s-shared-lib') _

// Define the Git repository URL and default branch
def gitUrl = 'https://github.com/naivedhagrawal/simple-java-maven-app-master.git'
def defaultBranch = 'main'

k8sPipeline([
    // Build stage that requires Docker functionality (uses dindPodTemplate.yaml)
    [
        name: 'Build',
        podImage: 'docker',  // Pod image to use for this stage
        podImageVersion: 'latest',  // Version of the image
        podTemplate: 'dindPodTemplate.yaml',  // Use the Docker-in-Docker pod template
        steps: [
            'echo "Building application with Docker..."',
            'docker -version'  // Build the Docker image
        ]
    ],
    // Test stage without Docker-in-Docker
    [
        name: 'Test',
        podImage: 'maven',
        podImageVersion: 'latest',
        // Default pod template (no dind)
        steps: [
            'echo "Running tests..."',
            'mvn -version'  // Run tests with shell script
        ]
    ],
    // Deploy stage without Docker-in-Docker
    [
        name: 'Deploy',
        podImage: 'python',
        podImageVersion: 'latest',
        steps: [
            'echo "Deploying application..."',
            'python -version'  // Deploy the application
        ]
    ]
], gitUrl, defaultBranch)
