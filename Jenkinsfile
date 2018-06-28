node {
  def app
  def dockerfile
  def anchorefile

  stage('Checkout') {
    // Clone the git repository
    checkout scm
    def path = sh returnStdout: true, script: "pwd"
    path = path.trim()
    dockerfile = path + "/Dockerfile"
    anchorefile = path + "/anchore_images"
  }

  stage('Build') {
    // Build the image and push it to a staging repository
    docker.withRegistry("https://index.docker.io/v1/", "dockerhub-wazowskis") {
      app = docker.build("wazowskis/foo:${BUILD_NUMBER}")
      app.push()
    }
  }

  stage('Parallel Test and Analyze') {
    parallel test: {
      app.inside {
          sh 'echo "Tests passed"'
      }
    },
    analyze: {
      def imagesLine = "docker.io/wazowskis/foo:${BUILD_NUMBER} " + dockerfile
      writeFile file: anchorefile, text: imagesLine
      anchore name: anchorefile, bailOnFail: false
    }
  }

  stage('Cleanup') {
    // Delete the docker image and clean up any allotted resources
    sh'''
      for i in `cat anchore_images | grep foo | awk '{print $1}'`;do docker rmi $i; done
    '''
  }
}
