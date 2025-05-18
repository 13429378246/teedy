pipeline {
 agent any

 environment {
 // define environment variable
// Jenkins credentials configuration
 DOCKER_HUB_CREDENTIALS = credentials('doc') // Docker Hub credentials ID store in Jenkins
 // Docker Hub Repository's name
DOCKER_IMAGE = 'panjianhui/teedy' // your Docker Hub user name and Repository's name
 DOCKER_TAG = "${env.BUILD_NUMBER}" // use build number as tag
  HTTP_PROXY  = "http://10.13.214.210:7890"
    HTTPS_PROXY = "http://10.13.214.210:7890"
    NO_PROXY    = "localhost,127.0.0.1"
   
    DOCKER_CLIENT_TIMEOUT = "300"
 }

 stages {
 stage('Build') {
 steps {
 checkout scmGit(
 branches: [[name: '*/master']],
 extensions: [],
 userRemoteConfigs: [[url: 'https://github.com/13429378246/teedy.git']]
// your github Repository
)
 sh 'mvn -B -DskipTests clean package'
 }
 }

 // Building Docker images
 stage('Building image') {
 steps {
 script {
 // assume Dockerfile locate at root
 docker.build("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}")
 }
 }
 }

stage('Upload image') {
  steps {
    script {
      // 从 Jenkins 凭证里取出 Docker Hub 的用户名和密码
      withCredentials([usernamePassword(
        credentialsId: 'doc',
        usernameVariable: 'DOCKER_USER',
        passwordVariable: 'DOCKER_PASS'
      )]) {
        sh '''
          # 延长 Docker CLI 超时，避免网络慢导致失败
          export DOCKER_CLIENT_TIMEOUT=300
          export COMPOSE_HTTP_TIMEOUT=300

          # 手动登录（等同于：docker login -u panjianhui -p pjh10086+ https://registry.hub.docker.com）
          echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin https://registry.hub.docker.com

          # Push 指定 tag
          docker push ${DOCKER_IMAGE}:${DOCKER_TAG}

          # 给 latest 打个 tag 并 push
          docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
          docker push ${DOCKER_IMAGE}:latest
        '''
      }
    }
  }
}


 // Running Docker container
 stage('Run containers') {
 steps {
 script {
 // stop then remove containers if exists
sh 'docker stop teedy-container-8081 || true'
sh 'docker rm teedy-container-8081 || true'
// run Container
docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").run(
'--name teedy-container-8081 -d -p 8081:8080'
)
// Optional: list all teedy-containers
sh 'docker ps --filter "name=teedy-container"'

}
}
 }
 }
}

