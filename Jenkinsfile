// Jenkinsfile at repo root (same level as Dockerfile and docker-compose.yml)
pipeline {
  agent { label 'miniserver' }

  stage('Prepare') {
    steps {
      script {
        env.GIT_HASH = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
        env.FULL_TAG = "${env.GIT_HASH}-${env.BUILD_NUMBER}"
      }
    }
  }
  
  environment {
    REGISTRY_URL   = 'nexus.server.cranie.com'
    REGISTRY_REPO  = 'ios'
    IMAGE_NAME     = 'calendar-app'  // Fixed: valid Docker image name
    REMOTE_REPO    = 'https://github.com/0507spc/Calendar-App.git'
    GIT_HASH       = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
    FULL_TAG       = "${env.GIT_HASH}-${env.BUILD_NUMBER}"
    BUILD_TAG      = "${BUILD_NUMBER}"
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
            docker compose build --no-cache
          '''
        }
      }
    }




    
    
  stage('Tag Images for Nexus') {
    steps {
      sh '''
        # API
        docker tag calendar-api ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-api:${FULL_TAG}
        docker tag calendar-api ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-api:latest
  
        # WEB
        docker tag calendar-web ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-web:${FULL_TAG}
        docker tag calendar-web ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-web:latest
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
        # API
        docker push ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-api:${FULL_TAG}
        docker push ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-api:latest
  
        # WEB
        docker push ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-web:${FULL_TAG}
        docker push ${REGISTRY_URL}/${REGISTRY_REPO}/calendar-web:latest
      '''
    }
  }

  post {
    always {
      sh 'docker logout ${REGISTRY_URL} || true'
    }
  }
}
