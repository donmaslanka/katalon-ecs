stage('Run Katalon') {
  steps {
    sh '''
      set -euo pipefail
      
      # Test network connectivity first
      echo "Testing network connectivity..."
      docker run --rm "${FULL_IMAGE}" curl -I https://www.katalon.com || echo "Network test failed"
      
      # Now run Katalon
      docker run --rm \
        -v "$PWD:/workspace" \
        -w /workspace \
        -e KATALON_API_KEY="${KATALON_API_KEY}" \
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