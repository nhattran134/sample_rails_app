#!groovy
// ---------------------- Begin Pipeline ----------------------
node('EC2CloudAgent') {
  stages {
    checkout()
    build_and_test()
  }
}

// ---------------------- Begin Support Methods ----------------------
def checkout() {
  stage('Checkout') {
    deleteDir()
    configure_variables()
  }
}


def build_image() {
    stage("Build image") {
    withAWS(role:"arn:aws:iam::677700034553:role/EC2AdminInstanceRole", region:"ap-southeast-1") {
      // AWS ECR login
      sh """
      aws --region ap-southeast-1 ecr get-login-password \
      | docker login --username AWS --password-stdin arn:aws:iam::677700034553:role/EC2AdminInstanceRole}
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
