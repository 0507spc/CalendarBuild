pipeline {
  agent { label 'miniserver' }

  environment {
    REGISTRY_URL  = 'nexus.server.cranie.com'
    REGISTRY_REPO = 'docker'
    REMOTE_REPO   = 'https://github.com/0507spc/Calendar-App.git'
  }

  triggers {
    pollSCM('H 2 * * *')
  }

  options {
    timestamps()
  }

  stages {

    stage('Clone Repo') {
      steps {
        dir('Calendar-App') {
          git url: "${REMOTE_REPO}", branch: 'main'
        }
      }
    }

    stage('Prepare Tags') {
      steps {
        dir('Calendar-App') {
          script {
            env.GIT_HASH = sh(
              returnStdout: true,
              script: 'git rev-parse --short HEAD'
            ).trim()

            env.FULL_TAG = "${env.GIT_HASH}-${env.BUILD_NUMBER}"
          }
        }
      }
    }

    stage('Build Images') {
      steps {
        dir('Calendar-App') {
          sh '''
            TAG=${FULL_TAG} docker compose build --no-cache
          '''
        }
      }
    }

    stage('Login to Nexus') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'nexus-docker-creds',
          usernameVariable: 'NEXUS_USER',
          passwordVariable: 'NEXUS_PASS'
        )]) {
          sh '''
            echo "$NEXUS_PASS" | docker login ${REGISTRY_URL} -u "$NEXUS_USER" --password-stdin
          '''
        }
      }
    }


    stage('Push Images') {
      steps {
        dir('Calendar-App') {
          sh '''
            TAG=${FULL_TAG} docker compose push
          '''
        }
      }
    }

    stage('Tag Latest') {
      steps {
        sh '''
          docker tag ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-app-api:${FULL_TAG} ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-app-api:latest
          docker tag ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-app-web:${FULL_TAG} ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-app-web:latest

          docker push ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-app-api:latest
          docker push ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-app-web:latest
        '''
      }
    }


    stage('Deploy') {
        steps {
            // SSH command to deploy
            sh 'ssh nas "cd /volume2/docker/calendar-app ; sudo docker compose pull ; sudo docker compose up -d"'
        }
    }



  
  }


    post {
    always {
      sh 'docker logout ${REGISTRY_URL} || true'
    }
    success {
      withCredentials([
        string(credentialsId: 'Pushover', variable: 'PUSHOVER_TOKEN'),
        string(credentialsId: 'Pushover-User', variable: 'PUSHOVER_USER')
      ]) {
        sh '''
          curl -s \
            --form-string "token=${PUSHOVER_TOKEN}" \
            --form-string "user=${PUSHOVER_USER}" \
            --form-string "message=Build succeeded for calendar-app" \
            https://api.pushover.net/1/messages.json
        '''
      }
    }
    failure {
      withCredentials([
        string(credentialsId: 'Pushover', variable: 'PUSHOVER_TOKEN'),
        string(credentialsId: 'Pushover-User', variable: 'PUSHOVER_USER')
      ]) {
        sh '''
          curl -s \
            --form-string "token=${PUSHOVER_TOKEN}" \
            --form-string "user=${PUSHOVER_USER}" \
            --form-string "message=Build failed for calendar-app" \
            https://api.pushover.net/1/messages.json
        '''
      }
    }
  }
}
