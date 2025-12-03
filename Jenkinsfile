pipeline {
    agent any

    environment {
        APP_NAME        = "eureka-server"
        IMAGE_NAME      = "eureka-server"
        IMAGE_TAG       = "latest"
        FULL_IMAGE      = "${IMAGE_NAME}:${IMAGE_TAG}"
        CONTAINER_NAME  = "eureka-server"
        HOST_PORT       = "3150"
        CONTAINER_PORT  = "3150"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh './gradlew clean test'
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                  docker build -t ${FULL_IMAGE} .
                """
            }
        }

        stage('Deploy') {
            steps {
                sh """
                  # 기존 컨테이너 있으면 정지+삭제
                  if [ \$(docker ps -aq -f name=${CONTAINER_NAME}) ]; then
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                  fi

                  # 새 컨테이너 실행
                  docker run -d --name ${CONTAINER_NAME} \\
                    -p ${HOST_PORT}:${CONTAINER_PORT} \\
                    ${FULL_IMAGE}
                """
            }
        }
    }
}
