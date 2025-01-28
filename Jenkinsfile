@Library('k8s-shared-lib') _

// Define the Git repository URL once
def gitUrl = 'https://github.com/myorg/myproject.git'

k8sPipeline([
    // Build stage that requires Docker functionality (uses dindPodTemplate.yaml)
    [
        name: 'Build',
        branch: 'feature/build-optimization',  // Specify the branch for this stage
        podImage: 'my-build-image',  // Pod image to use for this stage
        podImageVersion: 'v1.0.0',  // Version of the image
        podTemplate: 'dindPodTemplate.yaml',  // Use the Docker-in-Docker pod template
        steps: [
            'echo "Building application with Docker..."',
            'docker build -t my-app .'  // Build the Docker image
        ]
    ],
    // Test stage without Docker-in-Docker
    [
        name: 'Test',
        branch: 'develop',  // Use a different branch
        podImage: 'my-test-image',
        podImageVersion: 'v1.1.0',
        // Default pod template (no dind)
        steps: [
            'echo "Running tests..."',
            './run-tests.sh'  // Run tests with shell script
        ]
    ],
    // Deploy stage without Docker-in-Docker
    [
        name: 'Deploy',
        podImage: 'my-deploy-image',
        podImageVersion: 'v2.0.0',
        steps: [
            'echo "Deploying application..."',
            './deploy.sh'  // Deploy the application
        ]
    ]
], gitUrl)
