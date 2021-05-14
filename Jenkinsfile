#!groovy

pipeline {
  agent any
  stages {
    stage('ECR') {
      steps {
        withAWS(role:"arn:aws:iam::677700034553:role/EC2AdminInstanceRole", region:"ap-southeast-1") {
        // AWS ECR login
        sh """
        aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 677700034553.dkr.ecr.ap-southeast-1.amazonaws.com
        """
        }
        echo 'Building...'
        sh 'docker build -t rails-test-ecs .'
        sh 'docker tag rails-test-ecs:latest 677700034553.dkr.ecr.ap-southeast-1.amazonaws.com/rails-test-ecs:${BUILD_NUMBER}'
        sh 'docker tag rails-test-ecs:latest 677700034553.dkr.ecr.ap-southeast-1.amazonaws.com/rails-test-ecs:latest'
        sh 'docker push 677700034553.dkr.ecr.ap-southeast-1.amazonaws.com/rails-test-ecs:latest'
        sh 'docker push 677700034553.dkr.ecr.ap-southeast-1.amazonaws.com/rails-test-ecs:${BUILD_NUMBER}'
      }
    }
    stage('EB Deploy')
    {
      agent{
          docker{
            image '677700034553.dkr.ecr.ap-southeast-1.amazonaws.com/rails-test-ecs:${BUILD_NUMBER}'
          }
        }
      steps {
        deploy_app_environment()
      }
    }
  }
}

def deploy_app_environment() {
    // Create Dockerrun.aws.json in the deploy dir
    dir('deploy') {
      writeFile file: 'Dockerrun.aws.json', text: getDockerrunFile()
    }
    // Create zip package containing the contents of the docker run file 
    sh "rm -f ${BUILD_NUMBER}.zip"
    zip zipFile: "${BUILD_NUMBER}.zip", dir: 'deploy', archive: true

    echo "Deploying"
      withAWS(role: "arn:aws:iam::677700034553:role/EC2AdminInstanceRole", region:"ap-southeast-1") {
        sh "bundle exec eb_deploy -p ${BUILD_NUMBER}.zip -e 'dev'"
      }
}
// Dockerrun.aws.json configurations for apps
// Splunk container is leverage for monitoring and agent versions are controlled by the splunk-docker-image pipeline
def getDockerrunFile() {
  return """
  {
    "AWSEBDockerrunVersion": 2,
    "containerDefinitions": [
      {
        "name": "test",
        "image": "677700034553.dkr.ecr.ap-southeast-1.amazonaws.com/rails-test-ecs:latest",
        "essential": true,
        "memory": 2048,
        "portMappings": [
          {
            "hostPort": 80,
            "containerPort": 3000
          }
        ],
        "mountPoints": [
          {
            "sourceVolume": "awseb-logs-test",
            "containerPath": "/app/log"
          }
        ]
      }
    ]
  }
  """
}
