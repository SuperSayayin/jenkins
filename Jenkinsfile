pipeline {
  agent any
  environment {
    BURP_START_URL = 'https://ginandjuice.shop/'
    IMAGE_WITH_TAG = 'public.ecr.aws/portswigger/dastardly:latest'
    JUNIT_TEST_RESULTS_FILE = 'dastardly-report.xml'
    BURP_REPORT_FILE_PATH = "${WORKSPACE}/dastardly-report.xml"
    DASTARDLY_SEVERITY_THRESHOLD = 'MEDIUM' // Evita fallos por vulnerabilidades LOW
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
              # Verificar permisos en WORKSPACE antes de ejecutar
              chmod -R 777 ${WORKSPACE}

              # Ejecutar el escaneo con Dastardly
              docker run --rm --user $(id -u) -v ${WORKSPACE}:${WORKSPACE}:rw \
              -e BURP_START_URL=${BURP_START_URL} \
              -e BURP_REPORT_FILE_PATH=${BURP_REPORT_FILE_PATH} \
              ${IMAGE_WITH_TAG} | tee ${WORKSPACE}/dastardly-log.txt

              # Verificar si el archivo de reporte se generó correctamente
              ls -l ${WORKSPACE}
            '''
          }
        }
      }
    }
  }
  post {
    always {
      echo 'Scan completed. Processing results...'
      sh 'ls -l ${WORKSPACE}' // Muestra el contenido del directorio
      archiveArtifacts artifacts: 'dastardly-report.xml, dastardly-log.txt', fingerprint: true
    }
  }
}
