#!groovy

VARS = [
  groupId             : 'unknown',
  artifactId          : 'unknown',
  version             : 'unknown',
  build_version       : 'unknown',
  commit_hash         : 'unknown',
  app_name            : 'unknown',
  team                : 'unknown',
  owner_email         : 'unknown',
  pipeline_repo       : 'http://gitlab/root/tappy_local',
  portfolio           : 'unknown',
  delivery_stream     : 'unknown',
  release_train       : 'unknown',
  component_id        : 'unknown',
  workload            : 'unknown',
  component           : 'unknown',
  slack_channel       : 'unknown',
  aws_region          : 'ap-southeast_1',
  host_port           : '80',
  container_port      : '8080',
  container_mem       : '2048',
  eb_instance_size    : 't3.medium',
  solution_stack      : '64bit Amazon Linux 2018.03 v2.26.0 running Multi-container Docker 19.03.13-ce (Generic)'
]

ENVIRONMENTS = [
  lowers: [
    account_id       : '312182769717',
    build_version    : 'unknown',
    aws_role_arn     : 'arn:aws:iam::312182769717:role/acct-managed/awsappmannp-dealshield-deploy',
    ecs_repo         : 'unknown',
    s3_backend_bucket: 'unknown'
  ]
]

// ---------------------- Begin Pipeline ----------------------
node('kitchensink') {
  wraps {
    checkout()
    build_image()
  }
}

// ---------------------- Begin Support Methods ----------------------
def checkout() {
  stage('Checkout') {
    deleteDir()
    configure_variables()
  }
}
def configure_variables() {
  ENVIRONMENTS.keySet().each() { account ->
    ENVIRONMENTS[account].build_version = "${new Date().format( 'yyyyMMdd' )}-${VARS.commit_hash}"
    ENVIRONMENTS[account].ecs_repo = "${ENVIRONMENTS[account].account_id}.dkr.ecr.${VARS.aws_region}.amazonaws.com"
  }
}

def build_image() {
    stage("Build image") {
        withAWS(role:"${ENVIRONMENTS[account].aws_role_arn}", region:"${VARS.aws_region}") {
      // AWS ECR login
      sh """
      aws --region ${VARS.aws_region} ecr get-login-password \
      | docker login --username AWS --password-stdin ${ENVIRONMENTS[account].ecs_repo}
      """
    }
    tag_prefix = "${ENVIRONMENTS[account].ecs_repo}/${item}-${account}"
            sh """
            docker build \
            -t ${tag_prefix}:${ENVIRONMENTS[account].build_version} \
            -f Dockerfile . \
            --build-arg version=${MODULES[item].version}

              docker tag ${tag_prefix}:${ENVIRONMENTS[account].build_version} ${tag_prefix}:latest
              docker push ${tag_prefix}:${ENVIRONMENTS[account].build_version}
              docker push ${tag_prefix}:latest
            """
    }

}
