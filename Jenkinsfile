def call(Map params) {
def qg = "";

def DEPLOY_BRANCH= 'dev';
def VERSION_NO = "";
def BASE_VERSION='1.1.11';
def APP_NAME='ar'; 
def AWS_ENV_LOWER='';
def DOMAIN_NAME = ".fundflo.ai";
def API_URL ="ar-apis-dev.uat.fundflo.ai";
def BUCKET_NAME ="fundflo-stage-ap-south-1-files";
def GENERATED_ID = "";
// @Library('shared-library')_

pipeline {
  agent any
  // tools {nodejs "node209"}
  options {
    disableConcurrentBuilds()
    timeout(time: 1, unit: 'HOURS')
    buildDiscarder(logRotator(numToKeepStr: '3'))
  }
  parameters{
    choice(name:'AWS_ENV', choices:['DEV','QA','UAT'],description:'From which environment do you want to deploy?')
    choice(name:'AWS_REGION', choices:['ap-south-1'],description:'From which region do you want to deploy?')
  }
  stages {
      stage('GET BUILD INFO & AUTHORIZATION INFO'){
              steps{
                script{
                  //  sh 'env'
                  //  echo "ENVIRONMENT:  ${env} "
                  def cause = currentBuild.getBuildCauses('hudson.model.Cause$UserIdCause')
                  BUILD_USER_ID = "${cause.userId}"
                  BUILD_USER_ID = BUILD_USER_ID.replace("[","").replace("]","").trim()
                  BUILD_USER_NAME = "${cause.userName}"
                  BUILD_USER_NAME = BUILD_USER_NAME.replace("[","").replace("]","").trim()
                  AWS_ENV_LOWER = "${params.AWS_ENV}"
                  AWS_ENV_LOWER = AWS_ENV_LOWER.toLowerCase()
                  BUCKET_NAME = "fundflo-${AWS_ENV_LOWER}-ap-south-1-files"
                  if(params.AWS_ENV == 'DEV'){
                    API_URL = "ar-apis-dev.uat.fundflo.ai"
                  }else if(params.AWS_ENV == 'QA'){
                    BUCKET_NAME = "fundflo-qa-ap-south-1-files"                      
                    API_URL = "ar-apis-qa.uat.fundflo.ai"
                    if(BUILD_USER_ID.equalsIgnoreCase('shankar') || BUILD_USER_ID.equalsIgnoreCase('rattan') || BUILD_USER_ID.equalsIgnoreCase('kumar') || BUILD_USER_ID.equalsIgnoreCase('tushar') || BUILD_USER_ID.equalsIgnoreCase('omkar') || BUILD_USER_ID.equalsIgnoreCase('sachin')){}else{
                    currentBuild.result = 'ABORTED'
                      echo "ABORTED BUILD...:  ${BUILD_USER_NAME} - ${BUILD_USER_ID} "
                    throw new RuntimeException("Authorization Issue")
                  }
                  }else if(params.AWS_ENV == 'UAT'){
                    BUCKET_NAME = "fundflo-stage-ap-south-1-files"
                    API_URL = "stage-api.fundflo.ai"
                  }else if(params.AWS_ENV == 'PROD'){
                    API_URL = "test-enterprise-api.fundflo.ai"
                    BUCKET_NAME = "fundflo-${AWS_ENV_LOWER}-ap-south-1-files"
                        echo "ENVIRONMENT.....:${params.AWS_ENV} "
                        echo "BUILD_USER_ID...:${BUILD_USER_ID} "
                      if(BUILD_USER_ID.equalsIgnoreCase('shankar') || BUILD_USER_ID.equalsIgnoreCase('rattan')|| BUILD_USER_ID.equalsIgnoreCase('kumar')){}else{
                      currentBuild.result = 'ABORTED'
                       echo "ABORTED BUILD...:  ${BUILD_USER_NAME} - ${BUILD_USER_ID} "
                      throw new RuntimeException("Authorization Issue")
                    }
                  }
                  echo "API_URL.....:${API_URL} "
                  echo "BUCKET_NAME...:${BUCKET_NAME} "
                }
              }
          }
      stage('CREATE & PUSH TAG'){
          when {
            expression {env.BRANCH_NAME == DEPLOY_BRANCH }
          }
          steps{
            script{
                VERSION_NO =  gitTag "${BASE_VERSION}";
                GENERATED_ID = VERSION_NO.replace(".","");
            }
          }
    }
    stage('UPDATE ENVIRONMENT FILE'){
          when {
            expression {env.BRANCH_NAME == DEPLOY_BRANCH }
          }
          steps{
            script{
                  sh  "sed -e  's/%GENERATED_ID%/${GENERATED_ID}/g;s/%API_URL%/${API_URL}/g;s/%BUCKET_NAME%/${BUCKET_NAME}/g;' src/environments/environment_generate.ts > src/environments/environment.prod.ts"
                  sh  "sed -e  's/%GENERATED_ID%/${GENERATED_ID}/g;s/%API_URL%/${API_URL}/g;s/%BUCKET_NAME%/${BUCKET_NAME}/g;' src/environments/environment_generate.ts > src/environments/environment.ts"
            }
          }
    }
    stage('NPM INSTALL'){
        when {
          expression {env.BRANCH_NAME == DEPLOY_BRANCH }
        }
      steps{
        script{
          sh "npm cache clean --force"
          sh 'rm -rf package-lock.json && npm install'
        }
      }
    }
    stage('UPDATE THE BUILD NUMBER'){
      when {
        expression {env.BRANCH_NAME == DEPLOY_BRANCH }
      }
      steps {
        updateBuildNumber("${VERSION_NO}")
      }
    }
    stage('COMPILE AND GENERATE DIST'){
      when {
        expression {env.BRANCH_NAME == DEPLOY_BRANCH }
      }
      steps{
        script{
          // sh "npm run ng build --subresource-integrity --configuration=${AWS_ENV_LOWER}"
          // sh " npm run ng build --prod --aot --output-hashing=all"
           sh "ng build --configuration production --aot"
        }
      }
    }
  // stage('RUN UNIT TEST'){
  //     when {
  //       expression { params.AWS_ENV == 'DEV' && params.BUILD_AND_DEPLOY != 'deploy' && env.BRANCH_NAME == 'dev'}
  //     }
  //     steps{
  //       script{
  //           try{
  //             sh "npm run coverage"
  //           }catch(ex){
  //             echo ex.message
  //           }
  //           publishHTML target: [
  //             allowMissing: true,
  //             alwaysLinkToLastBuild: true,
  //             keepAll: false,
  //             reportDir: 'coverage/account-payable-app',
  //             reportFiles: 'index.html',
  //             reportName: 'AR-APP-Report'
  //           ]
  //         }
  //       }
  // }
    stage('DEPLOY TO S3') {
          when {
            expression {env.BRANCH_NAME == DEPLOY_BRANCH }
          }
      steps {
        script {
                withAWS(region:"${params.AWS_REGION}",credentials:"${params.AWS_ENV}"){
                    sh "aws s3 sync dist/. s3://fundflo-${AWS_ENV_LOWER}-${APP_NAME}-${params.AWS_REGION}"
                    distributionID = sh(script: 'aws cloudfront list-distributions --query "DistributionList.Items[*].{Domain: join(\', \', Aliases.Items), DistributionID: Id}[?contains(Domain, \''+"${APP_NAME}.${AWS_ENV_LOWER}"+"${DOMAIN_NAME}"+'\')] | [0].DistributionID" | tr -d \'"\' | tr -d \'\\n\'', returnStdout: true)
                      if(distributionID!=null){
                          try{
                            def INVALIDATION = sh (
                                  returnStdout: true,
                                  script:  "aws cloudfront create-invalidation --distribution-id ${distributionID} --paths '/*' ")
                          }
                          catch(e)
                          {
                            echo "Cloudfront invalidateion invoke exception:" + e
                          }
                      }else {
                        echo 'Distribution Id is null'
                      }
              }
        }
      }
    }
    stage('CLEAN UP'){
      steps{
              sh "rm -rf node_modules"
              sh "rm -rf package-lock.json"
      }
    }
  }
  post {
      success {
        slackSend botUser: true,
          channel: '#jenkinssuccess',
          color: '#095E0B',
          message: "SUCCESSFULLY BUILD BY : ${BUILD_USER_NAME}(${BUILD_USER_ID}) WITH JOB  ${env.JOB_NAME}(BUILD-NO: ${env.BUILD_NUMBER}) (<${env.BUILD_URL}|JENKINS-LINK>) -- (<https://${APP_NAME}.${AWS_ENV_LOWER}${DOMAIN_NAME}|API-LINK>)",
          tokenCredentialId: 'slack-token'
      }
      failure {
        slackSend botUser: true,
          channel: '#jenkinsfailure',
          color: '#F50A0A',
          message: "FAILED BUILD..... ${BUILD_USER_NAME}|${BUILD_USER_ID} - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)",
          tokenCredentialId: 'slack-token'
      }
      aborted {
        slackSend botUser: true,
        channel: '#jenkinsfailure',
        color: '#939693',
          message: "ABORTED BUILD..... ${BUILD_USER_NAME}|${BUILD_USER_ID} - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)",
        tokenCredentialId: 'slack-token'
        }
  }
}
}
