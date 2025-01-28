@Library('k8s-shared-lib') _

def gitUrl = 'https://github.com/naivedhagrawal/simple-java-maven-app-master.git'
def defaultBranch = 'main'

k8sPipeline([
    [
        name: 'Build',
        podImage: 'docker', // This is still needed even with DinD, for the agent container
        podImageVersion: 'latest', // Version for the agent container
        podTemplate: 'dindPodTemplate.yaml',
        steps: [
            'echo "Building application with Docker..."',
            'docker version',
            'docker build -t my-java-app .',
            'docker save my-java-app -o my-java-app.tar' // If you need to save the image
        ]
    ],
    [
        name: 'Test',
        podImage: 'maven',  // Required
        podImageVersion: 'latest', // Required
        steps: [
            'echo "Running tests..."',
            'mvn -version',
            'mvn test'
        ]
    ],
    [
        name: 'Deploy',
        podImage: 'python', // Required
        podImageVersion: 'latest', // Required
        steps: [
            'echo "Deploying application..."',
            'python -version' // Replace with your deployment script
        ]
    ]
], gitUrl, defaultBranch)