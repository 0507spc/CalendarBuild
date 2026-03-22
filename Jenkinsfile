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
            docker compose build --no-cache
          '''
        }
      }
    }

    stage('Tag Images') {
      steps {
        sh '''
          # API
          docker tag calendar-app-api ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-app-api:${FULL_TAG}
          docker tag calendar-app-api ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-app-api:latest

          # WEB
          docker tag calendar-app-web ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-app-web:${FULL_TAG}
          docker tag calendar-app-web ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-app-web:latest
        '''
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
        sh '''
          # API
          docker push ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-app-api:${FULL_TAG}
          docker push ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-app-api:latest

          # WEB
          docker push ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-web:${FULL_TAG}
          docker push ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-web:latest
        '''
      }
    }
  }

  post {
    always {
      sh 'docker logout ${REGISTRY_URL} || true'
    }
  }
}
