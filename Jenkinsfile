pipeline {
  agent any
  options {
    timestamps()
    ansiColor('xterm')
    buildDiscarder(logRotator(numToKeepStr: '20'))
  }
  triggers {
    // optional: run on every push if you later add a webhook
    // pollSCM('H/5 * * * *')
  }
  stages {
    stage('Preparation') {
      steps {
        sh '''
          set -euxo pipefail
          # Clean up any previous run so the build is idempotent
          docker stop samplerunning || true
          docker rm samplerunning || true
        '''
      }
    }

    stage('Build') {
      steps {
        sh '''
          set -euxo pipefail
          chmod +x ./sample-app.sh || true
          ./sample-app.sh
        '''
      }
    }

    stage('Test (acceptance)') {
      steps {
        sh '''
          set -euxo pipefail

          # Resolve container IPs dynamically so we don't hardcode them
          APP_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' samplerunning)
          JENKINS_IP=$(hostname -I | awk '{print $1}')

          echo "App IP: $APP_IP"
          echo "Jenkins IP: $JENKINS_IP"

          OUT=$(curl -s "http://$APP_IP:5050/")
          echo "Response from app:"
          echo "$OUT"

          # Verify the response contains the Jenkins container's IP
          echo "$OUT" | grep "You are calling me from $JENKINS_IP"
        '''
      }
    }
  }
  post {
    always {
      archiveArtifacts artifacts: '**/build.log', allowEmptyArchive: true
    }
    failure {
      echo 'Build failed. Check Console Output for details.'
    }
    success {
      echo 'Build + Test succeeded. App should be running on port 5050.'
    }
  }
}
