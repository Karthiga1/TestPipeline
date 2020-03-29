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
    stage('Check if stack is existing') {
      steps {
        withAWS(credentials:"${params.AWS_CREDENTIALS_ID}", region:"${params.REGION}") {
          dir ("${params.WORKING_DIR}") {
            // Check if Stack is existing
            script{
            def Stacks = sh "aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE --output text|grep myteststack"
            echo "${Stacks}"
            echo "0"
            if(Stacks.isEmpty()){
              def setStack = 'true'
              echo "1"

            }else{
              println("Stack already existing"); 
              echo "2"
            }
            }
          }
      }
     }
    }
    stage('Create Stack') {
      when {
                expression {setStack == 'true'}
            }
      steps {
        withAWS(credentials:"${params.AWS_CREDENTIALS_ID}", region:"${params.REGION}") {
          dir ("${params.WORKING_DIR}") {
             // Create Stack
             sh "aws cloudformation create-stack \
                --stackname ${params.STACK_NAME} \
                --template-url ${params.TEMPLATE_FILE_PATH} \
                ${params.EXTRA_ARGS} --capabilities CAPABILITY_NAMED_IAM"
  
             // Wait until Stack is created completely
             sh "aws cloudformation wait stack-create-complete \
                --stack-name ${params.STACK_NAME}" 
  
             // Print CloudFormation create command resutls
             sh "aws cloudformation describe-stacks \
                --stack-name ${params.STACK_NAME}"
          }
        }
      }
    }
    stage('Update Stack') {
      when {
                expression {setStack != 'true'}
            }
      steps {
        withAWS(credentials:"${params.AWS_CREDENTIALS_ID}", region:"${params.REGION}") {
          dir ("${params.WORKING_DIR}") {
             // Update Stack
             sh "aws cloudformation update-stack \
                --stackname ${params.STACK_NAME} \
                --template-url ${params.TEMPLATE_FILE_PATH} \
                ${params.EXTRA_ARGS}"
  
             
          }
        }
      }
    }
  }
}