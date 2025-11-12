pipeline {
  agent any

  environment {
    DOCKER_HOST = "${env.DOCKER_HOST_INTERNAL}"   // Variable para usar Daemon desde Docker externo (Configurar env.var en Jenkins 1°)
    DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'   // ID de la credencial en Jenkins
    GITHUB_CREDENTIALS = 'porta-jenkins'   // ID que creaste antes
    IMAGE_NAME = "milod0k3r/end-to-end-jenkins"
    IMAGE_TAG = "${env.BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        // checkout scm utiliza las credenciales configuradas en el job (ver más abajo)
        checkout scm
      }
    }

    stage('Build') {
      steps {
        sh 'echo "Construyendo la imagen Docker..."'
        sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
      }
    }

    stage('Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
          sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
          sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
          sh "docker push ${IMAGE_NAME}:latest"
        }
      }
    }

    stage('Deploy') {
      steps {
          sh 'echo "Desplegando contenedor local..."'
          sh "docker stop monit-${IMAGE_TAG} || true"
          sh "docker rm monit-${IMAGE_TAG} || true"
          sh "docker run -d --rm -p 8081:3000 --name monit-${IMAGE_TAG} ${IMAGE_NAME}:${IMAGE_TAG} || true"
          sh "sleep 3; curl -f http://localhost:8081/health || echo 'Deploy check failed (verificar)'"
  }
}
  }

  post {
    success {
      echo 'Pipeline finalizado correctamente.'
    }
    failure {
      echo 'Pipeline falló. Revisar logs.'
    }
  }
}
