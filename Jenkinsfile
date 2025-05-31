pipeline {
  agent any

  environment {
    IMAGE = "fatihsyir/demo-app"
    TAG = "latest"
    DOCKER_CRED = "docker-hub"
    KUBECONFIG_CRED = "kubeconfig-dev" // Optional, bisa dihapus
    NAMESPACE = "default"
    HELM_RELEASE = "casestudy-jenkins1"
  }

  stages {
    stage('Checkout Source Code') {
      steps {
        git url: 'https://github.com/fathbsyr/casestudy-jenkins', branch: 'main'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          echo "🛠️ Building image ${IMAGE}:${TAG}..."
          def builtImage = docker.build("${IMAGE}:${TAG}")
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: "docker-hub",
          usernameVariable: 'USER',
          passwordVariable: 'PASS'
        )]) {
          script {
            echo "📦 Pushing image to DockerHub..."
            sh """
              echo "$PASS" | docker login -u "$USER" --password-stdin
              docker push ${IMAGE}:${TAG}
            """
          }
        }
      }
    }

    stage('Deploy to Kubernetes (Helm)') {
      steps {
        script {
          echo "🚀 Deploying to Kubernetes via Helm..."
          sh '''
            export KUBECONFIG=/var/jenkins_home/kubeconfig
            helm upgrade --install $HELM_RELEASE ./helm \
              --set image.repository=$IMAGE \
              --set image.tag=$TAG \
              --namespace $NAMESPACE --create-namespace
          '''
        }
      }
    }
  } // <--- INI YANG KAMU LUPA TADI

  post {
    success {
      echo "✅ Pipeline Sukses: Aplikasi berhasil dideploy ke Kubernetes"
    }
    failure {
      echo "❌ Pipeline Gagal: Cek log untuk mengetahui error"
    }
  }
}
