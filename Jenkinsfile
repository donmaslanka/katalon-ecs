pipeline {
  agent any
  options { timestamps() }

  environment {
    IMAGE_NAME = "katalon-runner"
    IMAGE_TAG  = "build-${BUILD_NUMBER}"
    FULL_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"

    // Workspace/test settings
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
            docker version
            
            # Build with API key as build arg for activation during image build
            docker build \
              --build-arg KATALON_API_KEY="${KATALON_API_KEY}" \
              -t "${FULL_IMAGE}" .
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

            # Debug: Verify credentials are populated
            echo "KATALON_API_KEY length: ${#KATALON_API_KEY}"
            echo "KATALON_ORG_ID length: ${#KATALON_ORG_ID}"
            
            # Network connectivity check to Katalon license server
            echo "Testing connectivity to Katalon license server..."
            curl -I https://license.katalon.com --max-time 10 || {
              echo "WARNING: Cannot reach license.katalon.com - check firewall/security groups"
            }

            docker run --rm \
              -v "$PWD:/workspace" \
              -w /workspace \
              -e KATALON_API_KEY \
              -e KATALON_ORG_ID \
              "${FULL_IMAGE}" \
              katalonc \
                -noSplash \
                -runMode=console \
                -apiKey="$KATALON_API_KEY" \
                -orgID="$KATALON_ORG_ID" \
                -projectPath="/workspace/${KATALON_PROJECT_PATH}" \
                -testSuitePath="${TEST_SUITE_PATH}" \
                -executionProfile="${EXEC_PROFILE}" \
                -browserType="${BROWSER}" \
                -retry=0 \
                -licenseRelease
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