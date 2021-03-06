#!/usr/bin/env groovy
import java.nio.file.Files
import java.nio.file.Paths
import groovy.json.JsonSlurperClassic
import groovy.json.JsonOutput

newManEnvironmentTemplate ="orgmigration.postman_environment.json.template"
newManCollection="orgmigration-pipeline.postman_collection.json"
after_export="after_export.json"
//orgmigration_orgid.json
newManEnvironmentPrefix="orgmigration"
newManEnvironment=""
 orgIdKey="orgId"
 orgId=""
 stage_name=""
 pod1_url=""
 pod2_url=""
 isPod1ProxyEnabled=false
 isPod2ProxyEnabled=false
 isParentMigrationIdSet=false
 isOrgMigrationFailed=false
 migrationId=""

/*
What to put in Config File?
---------------------------

status.polling.period.milli=25000
pod1.url=https://pod.ics.dev:444
pod2.url=https://pod.ics.dev:444
idsma.haproxy=https://ids.ics.dev
adminUserName=admin
adminPassword=password

*/

pipeline {
  
  agent { label params.jenkins_label}
   parameters {

        string (
            defaultValue: 'qamgmt',
            description: 'Jenkins label',
            name : 'jenkins_label',
            trim : true)
        string(name: 'orgId', defaultValue: '', description: 'OrgId of the org to be migrated')
        string(name: 'parentMigrationId', defaultValue: '', description: '(If Moving Suborgs, what is the parent org migid? )')
        string(name: 'configFileId', defaultValue: 'org_migration_config', description: 'Configuration File Id(File that contains urls, passwords)')
        booleanParam(name: 'includeSubOrgs', defaultValue: false, description: '(Want to migrate sub-org also?)')
     }
  stages {

    stage("Input Sanitization"){
      steps {
          script {
             if( params.orgId == null || params.orgId.trim().isEmpty()){
               println(" error");
             }
             if(params.configFileId == null || params.configFileId.trim().isEmpty()){
               println("configFileId validation error")
               error("configFileId validation error/ can't be null or empty")
             }

            if( !(params.parentMigrationId == null || params.parentMigrationId.trim().isEmpty())){
                isParentMigrationIdSet = true
            }

          }

      }
    }
    stage("Cleanup") {
         steps {
                sh "rm -f *.xml"
                sh "rm -f *.log"
                sh "rm -f ${after_export}"
         }
    }
    stage("Install Dependencies") {
         steps {
           sh "npm config ls"
           sh "node -v"
          sh "npm install "
         }
    }
   stage('Input Reader') {
            steps {
              //MyPropertiesConfig configFileId
              //org_migration_config
              configFileProvider([configFile(fileId: "${configFileId}", variable: 'configOptions')]) {
                 script {  
                parentMigrationId=params.parentMigrationId.trim();
                includeSubOrgs=params.includeSubOrgs
                orgId=params.orgId.trim();
                stage_name=orgId+" to pod:"
                newManEnvironment=newManEnvironmentPrefix+"_"+orgId+".json"
                other =""
                if( isParentMigrationIdSet ){
                  other=" --includeSubOrgs="+params.includeSubOrgs+" --parentMigrationId="+params.parentMigrationId
                } else {
                  other=" --includeSubOrgs="+params.includeSubOrgs
                }
                myargs=" --orgid="+orgId+" --options=${env.configOptions} --template="+newManEnvironmentTemplate+"  --output="+newManEnvironment+" "+other
                def res = sh(script:  "node process_template.js ${myargs}", returnStdout: true)
                println("res:"+res)
            }//script
            }//config
          } //steps
      } // stage
      stage("Platform Service version check on Source and Destination pod") {
         steps {
          echo "Platform Service Version Check"
          sh "newman run ${newManCollection} --folder platformversioncheck -d service.csv -e ${newManEnvironment}"
         }
    }
   stage("Registered service health check") {
         steps {
          echo "Registered Service Health Check"
          sh "newman run ${newManCollection} --folder registeredservicehealthcheck -e ${newManEnvironment}"
         }
    }
      stage("Sub-org Validations"){
        failFast true
        parallel {
            stage("ParentMigrationId Validate") {
               when { equals expected: true, actual: isParentMigrationIdSet }
                steps {
                  echo "parent Migration id validation .... "
                }
            }
            stage("Destination Validation") {
                steps {
                      echo "Destination Validation: ==> ${stage_name}"
                      sh "newman run ${newManCollection} --folder DestinationPodValidation -e ${newManEnvironment}   --reporters cli  -k"
                }
            }
            stage("SubOrgCaseValidation"){
             //  when { equals expected: false, actual: isParentMigrationIdSet }
              steps {
                echo "Running SubOrgCaseValidation"
                sh "newman run ${newManCollection} --folder SubOrgCaseValidation -e ${newManEnvironment} --reporters cli -k"
              }
            }
        }//parallel
      } // validations
      stage("Initiate-OrgMigration") {
        steps {
          echo "Running the pod to pod migration"
          sh "newman run ${newManCollection} --folder OrgMigration -e ${newManEnvironment} --reporters cli -k --export-environment ./${after_export}"
        }
      }
      stage("OrgMigration Status Check"){
             steps {
                 echo "Running the pod to pod migration"
                 sh "newman run ${newManCollection} --folder Status -e ${after_export} --reporters cli -k "
             }
      } 
         stage("Source Troubleshooting pod") {
         steps {
          echo "Migration_Details_Source_POD: ==> ${stage_name}"
          sh "newman run ${newManCollection} --folder Migration_details_Source -e ${after_export}   --reporters cli  -k"
         }
    }
       stage("Destination Troubleshooting pod") {
         steps {
          echo "Migration_Details_Destination_POD ==> ${stage_name}"
          sh "newman run ${newManCollection} --folder Migration_details_Destination -e ${after_export} --reporters cli -k"
         }
    }
  } // stages
     post {
     always {
        sh "node cleanup_passwords.js  --envFile=${newManEnvironment}"
        sh "node cleanup_passwords.js  --envFile=after_export.json"
     }
   } 
  } //pipeline
