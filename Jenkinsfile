pipeline {
  agent any
  environment {
    BURP_START_URL='https://ginandjuice.shop/'
    IMAGE_WITH_TAG='public.ecr.aws/portswigger/dastardly:latest'
    JUNIT_TEST_RESULTS_FILE='dastardly-report.xml'
    BURP_REPORT_FILE_PATH="${WORKSPACE}/dastardly-report.xml"
    DASTARDLY_SEVERITY_THRESHOLD='MEDIUM' // Evita que falle por vulnerabilidades LOW
  }
  stages {
    stage ("Docker Pull Dastardly from Burp Suite container image") {
      steps {
        sh 'docker pull ${IMAGE_WITH_TAG}'
      }
    }
    stage ("Docker run Dastardly from Burp Suite Scan") {
      steps {
        cleanWs()
        script {
          catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
            sh '''
              docker run --rm --user $(id -u) -v ${WORKSPACE}:${WORKSPACE}:rw \
              -e BURP_START_URL=${BURP_START_URL} \
              -e BURP_REPORT_FILE_PATH=${BURP_REPORT_FILE_PATH} \
              ${IMAGE_WITH_TAG}
            '''
          }
        }
      }
    }
  }
  post {
    always {
      echo 'Scan completed. Processing results...'
      archiveArtifacts artifacts: 'dastardly-report.xml', fingerprint: true
    }
  }
}
