pipeline {
  agent {
    label 'docker-agent'
  }

  environment {
    IMAGE_NAME = "jayyy48/flask-demo"
    REGISTRY_CREDS = "dockerhub-creds"
  }

  stages {

    stage("Checkout") {
      steps {
        checkout scm
      }
    }

    stage("Build & Push Image (DEV only)") {
      when {
        branch "dev"
      }
      steps {
        script {
          def commitSha = sh(
            script: "git rev-parse --short HEAD",
            returnStdout: true
          ).trim()

          def imageTag = "${IMAGE_NAME}:${commitSha}"

          sh """
            echo "Building image: ${imageTag}"
            docker build -t ${imageTag} .
          """

          withCredentials([usernamePassword(
            credentialsId: REGISTRY_CREDS,
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
          )]) {
            sh """
              echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
              docker push ${imageTag}
            """
          }

          sh "echo ${imageTag} > image.txt"
          archiveArtifacts artifacts: 'image.txt', fingerprint: true
        }
      }
    }

    stage("Reference Image (RELEASE branch)") {
      when {
        branch "release"
      }
      steps {
        copyArtifacts(
          projectName: env.JOB_NAME,
          selector: lastSuccessful(),
          filter: "image.txt"
        )

        script {
          def image = readFile("image.txt").trim()
          echo "🚀 RELEASE is using image: ${image}"
        }
      }
    }

    stage("Reference Image (MAIN branch)") {
      when {
        branch "main"
      }
      steps {
        copyArtifacts(
          projectName: env.JOB_NAME,
          selector: lastSuccessful(),
          filter: "image.txt"
        )

        script {
          def image = readFile("image.txt").trim()
          echo "🏭 PROD (main) is using the image: ${image} not rebuilding"
        }
      }
    }
  }
}
