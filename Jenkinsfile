pipeline {
    agent any

    environment {
        APP_NAME        = "eureka-server"

        // 로컬에서 빌드할 이미지 이름
        //IMAGE_NAME      = "eureka-server"

        // 레지스트리에 올릴 풀네임
        REGISTRY        = "ghcr.io"
        GH_OWNER        = "sparta-next-me"
        IMAGE_REPO      = "eureka-server"
        FULL_IMAGE      = "${REGISTRY}/${GH_OWNER}/${IMAGE_REPO}:latest"

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
             sh '''
               ./gradlew clean test --no-daemon
               ./gradlew bootJar --no-daemon
             '''
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
        stage('Deploy to K8s') {
            steps {
                // 1) kubectl 설치 (없으면)
                sh '''
                  if ! command -v kubectl >/dev/null 2>&1; then
                    echo "kubectl not found. installing..."
                    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                    chmod +x kubectl
                    mv kubectl /usr/local/bin/kubectl
                  fi
                '''

                // 2) k3s kubeconfig 사용해서 배포
                withCredentials([
                    file(credentialsId: 'k3s-kubeconfig', variable: 'KUBECONFIG_FILE')
                ]) {
                    sh '''
                      export KUBECONFIG=${KUBECONFIG_FILE}

                      echo "Applying eureka-server manifest to k3s..."
                      kubectl apply -f eureka-server.yaml -n next-me

                      echo "Rollout status for eureka-server:"
                      kubectl rollout status deployment/eureka-server -n next-me || true

                      echo "Current eureka-server pods:"
                      kubectl get pods -n next-me -l app=eureka-server
                    '''
                }
            }
        }

}
