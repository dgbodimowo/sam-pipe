pipeline {
  agent any
  environment {
    PIPELINE_USER_CREDENTIAL_ID = 'dolly-keys'
    SAM_TEMPLATE = 'template.yaml'
    SAM_BUCKET = 'sam-pipe'
    MAIN_BRANCH = 'main'
    TESTING_STACK_NAME = 'sam-app2'
    TESTING_PIPELINE_EXECUTION_ROLE = 'arn:aws:iam::440393989684:role/aws-sam-cli-managed-dev-pipe-PipelineExecutionRole-7MYYMG6C6ZK1'
    TESTING_CLOUDFORMATION_EXECUTION_ROLE = 'arn:aws:iam::440393989684:role/aws-sam-cli-managed-dev-p-CloudFormationExecutionR-1WYDLULNT3YP5'
    TESTING_ARTIFACTS_BUCKET = 'aws-sam-cli-managed-dev-pipeline-artifactsbucket-1smze1tpot7ac'
    // If there are functions with "Image" PackageType in your template,
    // uncomment the line below and add "--image-repository ${TESTING_IMAGE_REPOSITORY}" to
    // testing "sam package" and "sam deploy" commands.
    // TESTING_IMAGE_REPOSITORY = '0123456789.dkr.ecr.region.amazonaws.com/repository-name'
    TESTING_REGION = 'us-east-1'
    PROD_STACK_NAME = 'sam-app3'
    PROD_PIPELINE_EXECUTION_ROLE = 'arn:aws:iam::440393989684:role/aws-sam-cli-managed-stage-pi-PipelineExecutionRole-180B6D6AA0519'
    PROD_CLOUDFORMATION_EXECUTION_ROLE = 'arn:aws:iam::440393989684:role/aws-sam-cli-managed-stage-CloudFormationExecutionR-1L9EARK54CT48'
    PROD_ARTIFACTS_BUCKET = 'aws-sam-cli-managed-stage-pipelin-artifactsbucket-1cq9rz92xpjnf'
    // If there are functions with "Image" PackageType in your template,
    // uncomment the line below and add "--image-repository ${PROD_IMAGE_REPOSITORY}" to
    // prod "sam package" and "sam deploy" commands.
    // PROD_IMAGE_REPOSITORY = '0123456789.dkr.ecr.region.amazonaws.com/repository-name'
    PROD_REGION = 'us-east-1'
  }
  stages {
    // uncomment and modify the following step for running the unit-tests
    // stage('test') {
    //   steps {
    //     sh '''
    //       # trigger the tests here
    //     '''
    //   }
    // }

    stage('build-and-deploy-feature') {
      // this stage is triggered only for feature branches (feature*),
      // which will build the stack and deploy to a stack named with branch name.
     // when {
     //   branch 'feature*'
     // }
      //agent {
       // docker {
        //  image 'public.ecr.aws/sam/build-provided'
         // args '--user 0:0 -v /var/run/docker.sock:/var/run/docker.sock'
       // }
      //}
      steps {
        sh 'sam build'
        //sh 'python3 -m venv venv && venv/bin/pip install aws-sam-cli'
        //stash includes: '**/venv/**/*', name: 'venv'
        //unstash 'venv'
        //sh 'venv/bin/sam build'        
        withAWS(
            credentials: env.PIPELINE_USER_CREDENTIAL_ID,
            region: env.TESTING_REGION,
            role: env.TESTING_PIPELINE_EXECUTION_ROLE,
            roleSessionName: 'deploying-feature') {
          sh '''
            sam deploy --stack-name $(echo ${BRANCH_NAME} | tr -cd '[a-zA-Z0-9-]') \
              --capabilities CAPABILITY_IAM \
              --region ${TESTING_REGION} \
              --s3-bucket ${TESTING_ARTIFACTS_BUCKET} \
              --no-fail-on-empty-changeset \
              --role-arn ${TESTING_CLOUDFORMATION_EXECUTION_ROLE}
          '''
        }
      }
    }

    stage('build-and-package') {
     // when {
       // branch env.MAIN_BRANCH
     // }
      //agent {
       // docker {
         // image 'public.ecr.aws/sam/build-provided'
         // args '--user 0:0 -v /var/run/docker.sock:/var/run/docker.sock'
       // }
     // }
      steps {
        sh 'python3 -m venv venv && venv/bin/pip install aws-sam-cli'
        stash includes: '**/venv/**/*', name: 'venv'
        unstash 'venv'
        sh 'venv/bin/sam build'
        withAWS(
            credentials: env.PIPELINE_USER_CREDENTIAL_ID,
            region: env.TESTING_REGION,
            role: env.TESTING_PIPELINE_EXECUTION_ROLE,
            roleSessionName: 'testing-packaging') {
          sh '''
            sam package \
              --s3-bucket ${TESTING_ARTIFACTS_BUCKET} \
              --region ${TESTING_REGION} \
              --output-template-file packaged-testing.yaml
          '''
        }

        withAWS(
            credentials: env.PIPELINE_USER_CREDENTIAL_ID,
            region: env.PROD_REGION,
            role: env.PROD_PIPELINE_EXECUTION_ROLE,
            roleSessionName: 'prod-packaging') {
          sh '''
            sam package \
              --s3-bucket ${PROD_ARTIFACTS_BUCKET} \
              --region ${PROD_REGION} \
              --output-template-file packaged-prod.yaml
          '''
        }

        archiveArtifacts artifacts: 'packaged-testing.yaml'
        archiveArtifacts artifacts: 'packaged-prod.yaml'
      }
    }

    stage('deploy-testing') {
     // when {
       // branch env.MAIN_BRANCH
     // }
      //agent {
       // docker {
        //  image 'public.ecr.aws/sam/build-provided'
       // }
     // }
      steps {
        withAWS(
            credentials: env.PIPELINE_USER_CREDENTIAL_ID,
            region: env.TESTING_REGION,
            role: env.TESTING_PIPELINE_EXECUTION_ROLE,
            roleSessionName: 'testing-deployment') {
          sh '''
            sam deploy --stack-name ${TESTING_STACK_NAME} \
              --template packaged-testing.yaml \
              --capabilities CAPABILITY_IAM \
              --region ${TESTING_REGION} \
              --s3-bucket ${TESTING_ARTIFACTS_BUCKET} \
              --no-fail-on-empty-changeset \
              --role-arn ${TESTING_CLOUDFORMATION_EXECUTION_ROLE}
          '''
        }
      }
    }

    // uncomment and modify the following step for running the integration-tests
    // stage('integration-test') {
    //   when {
    //     branch env.MAIN_BRANCH
    //   }
    //   steps {
    //     sh '''
    //       # trigger the integration tests here
    //     '''
    //   }
    // }

    stage('deploy-prod') {
     // when {
       // branch env.MAIN_BRANCH
     // }
     // agent {
      //  docker {
       //   image 'public.ecr.aws/sam/build-provided'
       // }
     // }
      steps {
        // uncomment this to have a manual approval step before deployment to production
        // timeout(time: 24, unit: 'HOURS') {
        //   input 'Please confirm before starting production deployment'
        // }
        withAWS(
            credentials: env.PIPELINE_USER_CREDENTIAL_ID,
            region: env.PROD_REGION,
            role: env.PROD_PIPELINE_EXECUTION_ROLE,
            roleSessionName: 'prod-deployment') {
          sh '''
            sam deploy --stack-name ${PROD_STACK_NAME} \
              --template packaged-prod.yaml \
              --capabilities CAPABILITY_IAM \
              --region ${PROD_REGION} \
              --s3-bucket ${PROD_ARTIFACTS_BUCKET} \
              --no-fail-on-empty-changeset \
              --role-arn ${PROD_CLOUDFORMATION_EXECUTION_ROLE}
          '''
        }
      }
    }
  }
}
