// Jenkinsfile
@Library('k8s-shared-lib') _

def gitUrl = 'https://github.com/naivedhagrawal/simple-java-maven-app-master.git'
def defaultBranch = 'main'

k8sPipeline([
    [
        name: 'Build',
        podImage: 'docker',
        podImageVersion: 'latest',
        podTemplate: 'dindPodTemplate.yaml',
        steps: [
            'echo "Building application with Docker..."',
            'docker version',
            'docker build -t my-java-app .',
            'docker save my-java-app -o my-java-app.tar'
        ]
    ],
    [
        name: 'Test',
        podImage: 'maven',
        podImageVersion: 'latest',
        steps: [
            'echo "Running tests..."',
            'mvn -version',
            'mvn test'
        ]
    ],
    [
        name: 'Deploy',
        podImage: 'python',
        podImageVersion: 'latest',
        steps: [
            'echo "Deploying application..."',
            'python -version' // Replace with your deployment script
        ]
    ]
], gitUrl, defaultBranch)