def label = "worker-${UUID.randomUUID().toString()}"

podTemplate(label: label, containers: [
  containerTemplate(name: 'npm', image: 'node:carbon-jessie', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
  // containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.8.8', command: 'cat', ttyEnabled: true),
  // containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:latest', command: 'cat', ttyEnabled: true)
],
volumes: [
  hostPathVolume(mountPath: '/home/npm/.gradle', hostPath: '/tmp/npm/.gradle'),
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {
  node(label) {
    def myRepo = checkout scm
    def gitCommit = myRepo.GIT_COMMIT
    def gitBranch = myRepo.GIT_BRANCH
    def shortGitCommit = "${gitCommit[0..10]}"
    def previousGitCommit = sh(script: "git rev-parse ${gitCommit}~", returnStdout: true)
 
    stage('Test') {
      try {
        container('npm') {
          sh """
            npm install
            npm test
            """
        }
      }
      catch (exc) {
        println "Failed to test - ${currentBuild.fullDisplayName}"
        throw(exc)
      }
    }
    stage('Build') {
      container('gradle') {
        sh "gradle build"
      }
    }
    stage('Docker push'){
      withCredentials([usernamePassword(credentialsId: 'dockerlogin', passwordVariable: 'dockerpassword', usernameVariable: 'dockerusername')]) {
        sh """
          docker login -u ${dockerusername} -p ${dockerpassword}
          docker build -t ${dockerusername}/data-date-display:1.1 .
          docker push ${dockerusername}/data-date-display:1.1
        """
      }
    }
    stage('kub push'){
      withCredentials([usernamePassword(credentialsId: 'dockerlogin', passwordVariable: 'dockerpassword', usernameVariable: 'dockerusername')]) {
        sh """
          docker login -u ${dockerusername} -p ${dockerpassword}
          docker build -t ${dockerusername}/data-date-display:1.1 .
          kubectl run --image=${dockerusername}/data-date-display:1.1 date-app --port=8081
        """
      } 

    }
    // stage('Run helm') {
    //   container('helm') {
    //     sh "helm list"
    //   }
    // }
  }
}