#!groovy
// ---------------------- Begin Pipeline ----------------------
node {
    checkout()
    build_image()
}

// ---------------------- Begin Support Methods ----------------------
def checkout() {
  stage('Checkout') {
    deleteDir()
  }
}


def build_image() {
    stage("Build image") {
    withAWS(role:"arn:aws:iam::677700034553:role/EC2AdminInstanceRole", region:"ap-southeast-1") {
      // AWS ECR login
      sh """
      aws --region ap-southeast-1 ecr get-login-password \
      | docker login --username AWS --password-stdin 677700034553.dkr.ecr.ap-southeast-1.amazonaws.com
      """
    }
    sh """
    docker build \
    -t 677700034553.dkr.ecr.ap-southeast-1.amazonaws.com/rails-test-ecs/test:${BUILD_NUMBER} \
    -f Dockerfile .

    docker 677700034553.dkr.ecr.ap-southeast-1.amazonaws.com/rails-test-ecs/test:${BUILD_NUMBER} 677700034553.dkr.ecr.ap-southeast-1.amazonaws.com/rails-test-ecs/test:latest
    docker push 677700034553.dkr.ecr.ap-southeast-1.amazonaws.com/rails-test-ecs:${BUILD_NUMBER}
    docker push 677700034553.dkr.ecr.ap-southeast-1.amazonaws.com/rails-test-ecs:latest
    """
    }

}
