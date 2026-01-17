pipeline {
  agent { label 'docker' }
  options { timestamps() }

  environment {
    IMAGE_NAME = "katalon-runner"
    IMAGE_TAG  = "build-${BUILD_NUMBER}"
    FULL_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"

    // CHANGE THESE:
    KATALON_PROJECT_PATH = "."                  // folder (in repo) that contains your Katalon project
    TEST_SUITE_PATH      = "Test Suites/Smoke"  // your real suite path
    EXEC_PROFILE         = "default"
    BROWSER              = "Chrome"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build runner image') {
      steps {
        sh '''
          set -euo pipefail
          docker version
          docker build -t "${FULL_IMAGE}" .
        '''
      }
    }

    stage('Run Katalon') {
      steps {
        sh '''
          set -euo pipefail
          echo "Workspace: $PWD"

          docker run --rm \
            -v "$PWD:/workspace" \
            -w /workspace \
            "${FULL_IMAGE}" \
            katalonc \
              -projectPath="/workspace/${KATALON_PROJECT_PATH}" \
              -testSuitePath="${TEST_SUITE_PATH}" \
              -executionProfile="${EXEC_PROFILE}" \
              -browserType="${BROWSER}" \
              -retry=0
        '''
      }
    }
  }

  post {
    always {
      sh '''
        set +e
        docker image ls | head -n 30
        docker system prune -af
      '''
    }
  }
}
