node {
  def app
  def dockerfile
  def anchorefile
  def repotag

  try {
    stage('Checkout') {
      // Clone the git repository
      checkout scm
      def path = sh returnStdout: true, script: "pwd"
      path = path.trim()
      dockerfile = path + "/Dockerfile"
      anchorefile = path + "/anchore_images"
      repotag = "${DOCKER_REPOSITORY}:${BUILD_NUMBER}"
    }

    stage('Build') {
      // Build the image and push it to a staging repository
      docker.withRegistry("${DOCKER_REGISTRY_URL}", "docker-credentials") {
        app = docker.build(repotag)
        app.push()
      }
    }

    stage('Parallel') {
      parallel Test: {
        app.inside {
            sh 'echo "Dummy - tests passed"'
        }
      },
      Analyze: {
        writeFile file: anchorefile, text: "${DOCKER_REGISTRY_HOSTNAME}/${DOCKER_REPOSITORY}:${BUILD_NUMBER} " + dockerfile
        anchore annotations: [[key: 'added-by', value: 'jenkins']], name: 'anchore_images'
      }
    }
  } finally {
    stage('Cleanup') {
      // Delete the docker image and clean up any allotted resources
      sh script: "docker rmi " + repotag
    }
  }
}
