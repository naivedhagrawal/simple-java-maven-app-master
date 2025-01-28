@Library('k8s-shared-lib') _

k8sPipeline([
    [
        name: 'Build',
        gitUrl: 'https://github.com/myorg/myproject.git',
        branch: 'feature/build-optimization', // Specify branch (optional)
        podImage: 'my-build-image',
        podImageVersion: 'v1.0.0',
        steps: [
            'echo "Building application..."',
            './build.sh'
        ]
    ],
    [
        name: 'Test',
        gitUrl: 'https://github.com/myorg/myproject.git',
        branch: 'develop', // Specify a different branch
        podImage: 'my-test-image',
        podImageVersion: 'v1.1.0',
        steps: [
            'echo "Running tests..."',
            './run-tests.sh'
        ]
    ],
    [
        name: 'Deploy',
        gitUrl: 'https://github.com/myorg/myproject.git',
        // No branch specified; defaults to 'main'
        podImage: 'my-deploy-image',
        podImageVersion: 'v2.0.0',
        steps: [
            'echo "Deploying application..."',
            './deploy.sh'
        ]
    ]
])
