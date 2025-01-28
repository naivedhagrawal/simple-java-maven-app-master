@Library('k8s-shared-lib') _

def gitUrl = 'https://github.com/naivedhagrawal/simple-java-maven-app-master.git'
def defaultBranch = 'main'

k8sPipeline([
    [
        name: 'Build',
        podTemplate: 'dindPodTemplate.yaml', // Use the DinD template
        steps: [
            'echo "Building application with Docker..."',
            'docker version', // Corrected command
            'docker build -t my-java-app .', // Example build command
            'docker save my-java-app -o my-java-app.tar' // Example: save the image
        ]
    ],
    [
        name: 'Test',
        podImage: 'maven',
        podImageVersion: 'latest',
        steps: [
            'echo "Running tests..."',
            'mvn -version',
            'mvn test' // Example test command
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