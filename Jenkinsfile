podTemplate(
  agentContainer: 'jnlp',
  showRawYaml: false,
  containers: [
    containerTemplate(name: 'jnlp', image: 'jenkins/inbound-agent:latest', alwaysPullImage: true),
    containerTemplate(name: 'all-in-one', image: 'naivedh/jenkins-agent-all-in-one:latest', command: 'cat', ttyEnabled: true, alwaysPullImage: true)
  ]) 

{
  node('jnlp') {
    stage('Checkout') {
      checkout scm
    }
    stage('Build') {
      container('all-in-one') {
        sh 'mvn -B -DskipTests clean package'
      }
    }
  }
}