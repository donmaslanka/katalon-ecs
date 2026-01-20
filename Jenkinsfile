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
        sh '''
          set -euo pipefail
          docker version
          docker build -t "${FULL_IMAGE}" .
        '''
      }
    }

    stage('Run Katalon') {
      steps {
        // Bind secrets ONLY within this stage; requires both creds to be Jenkins "Secret text"
        withCredentials([
          string(credentialsId: 'katalon-api-key', variable: 'KATALON_API_KEY'),
          string(credentialsId: 'katalon-org-id',  variable: 'KATALON_ORG_ID')
        ]) {
          sh '''
            set -euo pipefail

            # Non-secret debugging to prove vars are populated (safe: prints only lengths)
            echo "KATALON_API_KEY length: ${#KATALON_API_KEY}"
            echo "KATALON_ORG_ID length: ${#KATALON_ORG_ID}"

            # Optional quick egress check (uncomment if you suspect NAT/DNS issues in ECS/private subnets)
            # curl -I https://license.katalon.com --max-time 10 || true

            docker run --rm \
              -v "$PWD:/workspace" \
              -w /workspace \
              -e KATALON_API_KEY \
              -e KATALON_ORG_ID \
              "${FULL_IMAGE}" \
              katalonc \
                -apiKey="$KATALON_API_KEY" \
                -orgID="$KATALON_ORG_ID" \
                -projectPath="/workspace/${KATALON_PROJECT_PATH}" \
                -testSuitePath="${TEST_SUITE_PATH}" \
                -executionProfile="${EXEC_PROFILE}" \
                -browserType="${BROWSER}" \
                -retry=0
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
