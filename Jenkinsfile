pipeline {
    agent any

    environment {
        APP_NAME        = "eureka-server"
        REGISTRY        = "ghcr.io"
        GH_OWNER        = "sparta-next-me"
        IMAGE_REPO      = "eureka-server"
        FULL_IMAGE      = "${REGISTRY}/${GH_OWNER}/${IMAGE_REPO}:latest"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh './gradlew clean bootJar --no-daemon'
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    sh "docker build -t ${FULL_IMAGE} ."
                    withCredentials([usernamePassword(credentialsId: 'ghcr-credential', usernameVariable: 'USER', passwordVariable: 'TOKEN')]) {
                        sh "echo \$TOKEN | docker login ${REGISTRY} -u \$USER --password-stdin"
                        sh "docker push ${FULL_IMAGE}"
                    }
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                withCredentials([file(credentialsId: 'k3s-kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                      # 1. 네임스페이스 생성 (없을 경우 대비)
                      kubectl create namespace next-me --dry-run=client -o yaml | kubectl apply -f -

                      # 2. YAML 적용
                      kubectl apply -f eureka-server.yaml -n next-me

                      # 3. 배포 확인
                      echo "Waiting for rollout..."
                      kubectl rollout status deployment/eureka-server -n next-me

                      # 4. 어느 노드에 떴는지 상세 확인
                      echo "Deployment location:"
                      kubectl get pods -n next-me -l app=eureka-server -o wide
                    '''
                }
            }
        }
    }
}