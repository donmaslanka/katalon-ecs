pipeline {
  agent any
  options { timestamps() }
  environment {
    IMAGE_NAME = "katalon-runner"
    IMAGE_TAG  = "build-${BUILD_NUMBER}"
    FULL_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"
    KATALON_PROJECT_PATH = "."
    TEST_SUITE_PATH      = "Test Suites/Smoke"
    EXEC_PROFILE         = "default"
    BROWSER              = "Chrome"
    KATALON_API_KEY      = credentials('katalon-api-key')
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
          
          docker run --rm \
            -v "$PWD:/workspace" \
            -w /workspace \
            -e KATALON_API_KEY="${KATALON_API_KEY}" \
            -e API_KEY="${KATALON_API_KEY}" \
            "${FULL_IMAGE}" \
            katalonc \
              -apiKey="${KATALON_API_KEY}" \
              -projectPath="/workspace/${KATALON_PROJECT_PATH}" \
              -testSuitePath="${TEST_SUITE_PATH}" \
              -executionProfile="${EXEC_PROFILE}" \
              -browserType="${BROWSER}" \
              -retry=0 \
              -noExit
        '''
      }
    }
  }
  post {
    always {
      sh '''
        set +e
        docker system prune -af
      '''
    }
  }
}