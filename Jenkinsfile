pipeline {
    agent any

    environment {
        APP_NAME        = "eureka-server"

        // 로컬에서 빌드할 이미지 이름
        //IMAGE_NAME      = "eureka-server"

        // 레지스트리에 올릴 풀네임
        IMAGE_NAME      = "ghcr.io/nextme/eureka-server"
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
                sh './gradlew bootJar'
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                  docker build -t ${FULL_IMAGE} .
                """
            }
        }

        // 이미지 레지스트리에 push
        stage('Push Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'ghcr-credential',   // Jenkins에 미리 만들어 둘 크리덴셜 ID
                        usernameVariable: 'REGISTRY_USER',
                        passwordVariable: 'REGISTRY_TOKEN'
                    )
                ]) {
                    sh """
                      echo "$REGISTRY_TOKEN" | docker login ghcr.io -u "$REGISTRY_USER" --password-stdin
                      docker push ${FULL_IMAGE}
                    """
                }
            }
        }

        //  지금은 EC2에 바로 띄우는 단계 (나중에 K8s 가면 이거만 교체 예정)
        stage('Deploy') {
            steps {
                sh """
                  # 기존 컨테이너 있으면 정지+삭제
                  if [ \$(docker ps -aq -f name=${CONTAINER_NAME}) ]; then
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                  fi

                  # 새 컨테이너 실행 (레지스트리에서 받아온 이미지로도 가능)
                  docker run -d --name ${CONTAINER_NAME} \\
                    -p ${HOST_PORT}:${CONTAINER_PORT} \\
                    ${FULL_IMAGE}
                """
            }
        }
    }
}
