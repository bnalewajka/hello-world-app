#!/env groovy

def label = "workshops-pod-${UUID.randomUUID().toString()}"
podTemplate(label: label,
    containers: [
        containerTemplate(
            name: 'docker',
            image: 'docker:1.12',
            resourceRequestMemory: '250Mi',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'jdk',
            image: 'openjdk:8-alpine',
            resourceRequestMemory: '250Mi',
            ttyEnabled: true,
            command: 'cat'
        )],
        volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]
) {
  node(label) {
    stage('Assemble') {
      container('jdk') {
        checkout scm
        sh "./gradlew clean assemble"
      }
    }
    stage('Test') {
      try {
        container('jdk') {
          sh "./gradlew test"
        }
      } catch (Exception ignored) {
        currentBuild.result = "FAILURE"
      }
    }
    stage('Report') {
      junit allowEmptyResults: true, testResults: 'build/test-results/**/*.xml'
      def jdkLog = containerLog(name: 'jdk', returnLog: true)
      writeFile encoding: 'UTF-8', file: 'build.log', text: jdkLog
      archiveArtifacts allowEmptyArchive: true, artifacts: 'build.log'
    }
    stage('Image') {
        if (env.BRANCH_NAME == "master") {
            container('jdk') {
                  sh "./gradlew dockerPushImage -PbuildNumber=${env.BUILD_NUMBER} -PdockerUsername=bartosznalewajka -PdockerPassword=jamf1234!"
            }
        }
    }
  }
}