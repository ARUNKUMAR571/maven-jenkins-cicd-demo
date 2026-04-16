pipeline {
  agent any

  environment {
    IMAGE_NAME = "theshubhamgour/maven-jenkins-demo"
  }

  stages {

    stage('Checkout') {
      steps {
        echo "Checking out code from SCM..."
        checkout scm
      }
    }

    stage('Clean Workspace') {
      steps {
        echo "Cleaning previous build files..."
        sh 'mvn clean'
      }
    }

    stage('Build') {
      steps {
        echo "Building Maven project..."
        sh 'mvn compile'
      }
    }

    stage('Unit Tests') {
      steps {
        echo "Running unit tests..."
        sh 'mvn test'
      }
      post {
        always {
          script {
            if (fileExists('target/surefire-reports')) {
              echo "Publishing JUnit test results..."
              junit 'target/surefire-reports/*.xml'
            } else {
              echo "No JUnit test report directory found — skipping test report publishing."
            }
          }
        }
      }
    }

    stage('Package') {
      steps {
        echo "Packaging application into JAR..."
        sh 'mvn package -DskipTests'
      }
      post {
        success {
          archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
      }
    }

    stage('Static Code Analysis (Optional)') {
      steps {
        echo "Running code analysis (Simulated)..."
        sh 'echo "Code quality check passed ✅"'
      }
    }

    stage('Build Docker Image') {
      steps {
        echo "Building Docker image..."
        sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
      }
    }

    stage('DockerHub Login & Push') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'DockerHub',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {

          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push '"$IMAGE_NAME"':'"$BUILD_NUMBER"'
            docker logout
          '''
        }
      }
    }

    stage('Deploy (Run Container)') {
      steps {
        echo "Deploying Docker container..."
        sh '''
          docker stop maven-demo || true
          docker rm maven-demo || true

          docker run -d --name maven-demo '"$IMAGE_NAME"':'"$BUILD_NUMBER"'

          sleep 3
          docker logs maven-demo
        '''
      }
    }

    stage('Cleanup') {
      steps {
        echo "Cleaning up Docker resources..."
        sh '''
          docker stop maven-demo || true
          docker rm maven-demo || true
          docker rmi '"$IMAGE_NAME"':'"$BUILD_NUMBER"' || true
        '''
      }
    }
  }

 post {
    success {
        echo "✅ Build ${env.BUILD_NUMBER} completed successfully!"
    }
    failure {
        echo "❌ Build ${env.BUILD_NUMBER} failed!"
    }
    always {
        cleanWs()
    }
}
}
