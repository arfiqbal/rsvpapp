def label = "worker-${UUID.randomUUID().toString()}"

podTemplate(label: label, containers: [
  containerTemplate(name: 'docker', image: 'docker:18.09', command: 'cat', ttyEnabled: true, envVars: [envVar(key: 'DOCKER_HOST', value: '127.0.0.1')]),
  containerTemplate(name: 'dind', image: 'docker:18.09-dind', ttyEnabled: true, privileged: true)
],
volumes: [
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {
  node(label) {
    def myRepo = checkout scm
    def gitCommit = myRepo.GIT_COMMIT
    def gitBranch = myRepo.GIT_BRANCH
    def shortGitCommit = "${gitCommit[0..10]}"
    def previousGitCommit = sh(script: "git rev-parse ${gitCommit}~", returnStdout: true)

    stage('Create Docker images') {
      container('docker') {
        withCredentials([[$class: 'UsernamePasswordMultiBinding',
          credentialsId: 'dockerhub',
          usernameVariable: 'DOCKER_HUB_USER',
          passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
          sh """
            docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD}
            docker build -t nkhare/rsvp-demo:${gitCommit} .
            docker push nkhare/rsvp-demo:${gitCommit}
            """
        }
      }
    }
    stage('Run helm') {
      container('helm') {
        sh "helm list"
      }
    }
  }
}
