// Jenkinsfile at repo root (same level as Dockerfile and docker-compose.yml)
pipeline {
  agent { label 'miniserver' }
  
  environment {
    REGISTRY_URL   = 'nexus.server.cranie.com'
    REGISTRY_REPO  = 'ios'
    IMAGE_NAME     = 'calendar-app'  // Fixed: valid Docker image name
    REMOTE_REPO   = 'https://github.com/0507spc/Calendar-App.git'
    GIT_HASH       = "${sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()}"
    BUILD_TAG      = "${BUILD_NUMBER}"
    FULL_TAG       = "${GIT_HASH}-${BUILD_TAG}"
    DOCKER_IMAGE   = "${REGISTRY_URL}/${REGISTRY_REPO}/${IMAGE_NAME}:${FULL_TAG}"
    LATEST_IMAGE   = "${REGISTRY_URL}/${REGISTRY_REPO}/${IMAGE_NAME}:latest"
  }

  triggers {
    pollSCM('H 2 * * *')
  }

  options {
    timestamps()
  }

  stages {
    stage('Clone Remote Repo') {
      steps {
        dir('Calendar-App') {  // Clone into subdir
          git url: "${REMOTE_REPO}", branch: 'main'  // or 'main'
        }
      }
    }

    
    stage('Docker Build') {
      steps {
        dir('Calendar-App') {
          sh '''
            docker build --no-cache -t ${IMAGE_NAME}:${FULL_TAG} .
          '''
        }
      }
    }




    
    
    stage('Tag Images for Nexus') {
      steps {
        sh '''
          docker tag ${IMAGE_NAME}:${FULL_TAG} "${DOCKER_IMAGE}"
          docker tag ${IMAGE_NAME}:${FULL_TAG} "${LATEST_IMAGE}"
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

    stage('Push to Nexus') {
      steps {
        sh '''
          docker push "${DOCKER_IMAGE}"
          docker push "${LATEST_IMAGE}"
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
