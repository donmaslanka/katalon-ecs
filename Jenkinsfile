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
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build runner image') {
      steps {
        withCredentials([
          string(credentialsId: 'katalon-api-key', variable: 'KATALON_API_KEY')
        ]) {
          sh '''
            set -euo pipefail
            
            echo "Building Katalon runner image..."
            echo "API key length: ${#KATALON_API_KEY}"
            
            docker build \
              --build-arg KATALON_API_KEY="${KATALON_API_KEY}" \
              -t "${FULL_IMAGE}" .
          '''
        }
      }
    }

    stage('Debug Container Environment') {
      steps {
        withCredentials([
          string(credentialsId: 'katalon-api-key', variable: 'KATALON_API_KEY'),
          string(credentialsId: 'katalon-org-id',  variable: 'KATALON_ORG_ID')
        ]) {
          sh '''
            set -euo pipefail
            
            echo "=== HOST ENVIRONMENT CHECK ==="
            echo "KATALON_API_KEY length: ${#KATALON_API_KEY}"
            echo "KATALON_ORG_ID length: ${#KATALON_ORG_ID}"
            echo "First 10 chars of API key: ${KATALON_API_KEY:0:10}"
            
            echo ""
            echo "=== CONTAINER ENVIRONMENT CHECK ==="
            docker run --rm \
              -e KATALON_API_KEY \
              -e KATALON_ORG_ID \
              "${FULL_IMAGE}" \
              bash -c 'echo "Inside container - API key length: ${#KATALON_API_KEY}"; echo "Inside container - ORG ID length: ${#KATALON_ORG_ID}"; echo "API key from ENV: ${KATALON_API_KEY:0:10}"; echo "API key from build ARG-based ENV: $KATALON_API_KEY" | cut -c1-10'
          '''
        }
      }
    }

    stage('Run Katalon') {
      steps {
        withCredentials([
          string(credentialsId: 'katalon-api-key', variable: 'KATALON_API_KEY'),
          string(credentialsId: 'katalon-org-id',  variable: 'KATALON_ORG_ID')
        ]) {
          sh '''
            set -euo pipefail

            echo "=== Network Connectivity Test ==="
            curl -I https://api.katalon.com || echo "WARNING: Cannot reach api.katalon.com"

            echo "=== Running Katalon Tests ==="
            docker run --rm \
              -v "$PWD:/workspace" \
              -w /workspace \
              -e KATALON_API_KEY \
              -e KATALON_ORG_ID \
              "${FULL_IMAGE}" \
              bash -lc "katalonc -noSplash -runMode=console -apiKey=\"\${KATALON_API_KEY}\" -orgID=\"\${KATALON_ORG_ID}\" -projectPath=\"/workspace/${KATALON_PROJECT_PATH}\" -testSuitePath=\"${TEST_SUITE_PATH}\" -executionProfile=\"${EXEC_PROFILE}\" -browserType=\"${BROWSER}\" -retry=0 -licenseRelease"
          '''
        }
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
EOF