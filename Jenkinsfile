@Library('k8s-shared-lib') _

k8sPipeline([
    [
        name: 'Build',
        gitUrl: 'https://github.com/myorg/myproject.git',
        branch: 'feature/build-optimization',
        podImage: 'my-build-image',
        podImageVersion: 'v1.0.0',
        podTemplate: 'dindPodTemplate.yaml', // Use the dind pod template
        steps: [
            'echo "Building application with Docker..."',
            'docker build -t my-app .'
        ]
    ],
    [
        name: 'Test',
        gitUrl: 'https://github.com/myorg/myproject.git',
        branch: 'develop',
        podImage: 'my-test-image',
        podImageVersion: 'v1.1.0',
        // Default pod template (no dind)
        steps: [
            'echo "Running tests..."',
            './run-tests.sh'
        ]
    ],
    [
        name: 'Deploy',
        gitUrl: 'https://github.com/myorg/myproject.git',
        podImage: 'my-deploy-image',
        podImageVersion: 'v2.0.0',
        steps: [
            'echo "Deploying application..."',
            './deploy.sh'
        ]
    ]
])
