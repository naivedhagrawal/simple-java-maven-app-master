podTemplate(
  agentContainer: 'jnlp',
  agentInjection: true,
  showRawYaml: false,
  containers: [
    containerTemplate(name: 'jnlp', image: 'jenkins/inbound-agent:latest', alwaysPullImage: true, privileged: true),
    containerTemplate(name: 'all-in-one', image: 'naivedh/jenkins-agent-all-in-one:latest',alwaysPullImage: true, command: 'cat', ttyEnabled: true)
  ])

{
  node(POD_LABEL) {
    stage('Checkout') {
      checkout scm
    }
    stage('Build') {
      container('all-in-one') {
        sh 'mvn -B -DskipTests clean package'
      }
    }
    stage('test') {
      container('all-in-one') {
        sh 'mvn test'
      }
    }
    stage('Deliver') {
      container('all-in-one'){
            steps {
                sh './jenkins/scripts/deliver.sh'
            }
      }
    }
  }
}