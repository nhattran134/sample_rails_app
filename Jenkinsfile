#!groovy

pipeline {
  agent any
  stages {
    stage('ECR') {
      steps {

        sh 'aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 677700034553.dkr.ecr.ap-southeast-1.amazonaws.com'
        echo 'Building...'
        sh 'docker build -t rails-test-ecs .'
        sh 'docker tag rails-test-ecs:latest 677700034553.dkr.ecr.ap-southeast-1.amazonaws.com/rails-test-ecs:latest'
        sh 'docker push 677700034553.dkr.ecr.ap-southeast-1.amazonaws.com/rails-test-ecs:latest'
      }
    }
  }
}

def deploy() {
  def creds = readJSON file: './sts.json'
  withEnv(['AWS_ACCESS_KEY_ID='+creds.AWS_ACCESS_KEY_ID, 'AWS_SECRET_ACCESS_KEY='+creds.AWS_SECRET_ACCESS_KEY, 'AWS_SESSION_TOKEN='+creds.AWS_SESSION_TOKEN]) {
    sh "cd src/apps/orders && yarn deploy:$STAGE"
  }
}
