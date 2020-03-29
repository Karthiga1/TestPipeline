#!groovy

pipeline {

  agent any

  parameters {
    string(
      name: 'AWS_CREDENTIALS_ID',
      defaultValue: '',
      description: 'AWS credentials id, stored in Jenkins credentials'
    )
    string(
      name: 'EXTRA_ARGS',
      defaultValue: '',
      description: 'aws cloudformation create-stack command extra arguments'
    )
    string(
      name: 'GIT_BRANCHES_CFN',
      defaultValue: '*/master',
      description: "Git branch or tag name or commit id to retrieve of GIT_URL of CloudFormation template file"
    )
    string(
      name: 'GIT_URL',
      defaultValue: '',
      description: "GitHub URL to retrieve CloudFormation template"
    )
    string(
      name: 'REGION',
      defaultValue: '',
      description: 'AWS CLI region name'
    )
    string(
      name: 'STACK_NAME',
      defaultValue: '',
      description: 'CloudFormation stack name'
    )
    string(
      name: 'TEMPLATE_FILE_PATH',
      defaultValue: '',
      description: 'CloudFormation template file path'
    )
    string(
      name: 'WORKING_DIR',
      defaultValue: 'cfn',
      description: 'Job working directory'
    )
  }

  stages {
    stage('Initialize wokring directory') {
      steps {
        dir ("${params.WORKING_DIR}") {
          cleanWs()
        }
      }
    }

    stage('Retrieve CloudFormation template file from Github') {
      steps {
        checkout(
          [
            $class: 'GitSCM',
            branches: [
              [
                name: "${params.GIT_BRANCHES_CFN}"
              ]
            ],
            extensions: [
              [
                $class: 'RelativeTargetDirectory',
                relativeTargetDir: "${params.WORKING_DIR}"
              ]
            ],
            doGenerateSubmoduleConfigurations: false,
            submoduleCfg: [],
            userRemoteConfigs: [
              [
                url: "${params.GIT_URL}"
              ]
            ]
          ]
        )
      }
    }

    stage('Create Stack') {
      steps {
        withAWS(credentials:"${params.AWS_CREDENTIALS_ID}", region:"${params.REGION}") {
          dir ("${params.WORKING_DIR}") {
             // Create Stack
             sh "/usr/local/bin/aws cloudformation create-stack \
                --stack-name ${params.STACK_NAME} \
                --template-url ${params.TEMPLATE_FILE_PATH} \
                ${params.EXTRA_ARGS} --capabilities CAPABILITY_NAMED_IAM --debug"
  
             // Wait until Stack is created completely
             sh "/usr/local/bin/aws cloudformation wait stack-create-complete \
                --stack-name ${params.STACK_NAME}" 
  
             // Print CloudFormation create command resutls
             sh "/usr/local/bin/aws cloudformation describe-stacks \
                --stack-name ${params.STACK_NAME}"
          }
        }
      }
    }
  }
}
