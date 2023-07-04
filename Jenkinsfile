
import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}


environment {
        AZURE_SUBSCRIPTION_ID = 'ec940544-e7ec-4f93-ade7-8e607a0b020b'
        AZURE_TENANT_ID = '507fc407-e0b7-4432-be14-ffb107fb7e24'
        AZURE_CREDENTIALS_ID = 'da863f92-0a7e-4198-b7bb-be20e7b9dbca'
        AZURE_CLIENT_SECRET = 'J_S8Q~JNF7K-Bb-i6VPUcu1mshDiheJmA-em3ae6'
        AZURE_CLIENT_ID = 'd2beb018-8f93-4e2c-bfd4-bbf9535e4ef8'

    }

    stages{
    stage('init') {
        steps{
             checkout scm
        }
     
    }
  
    stage('build') {
        steps{
             sh 'mvn clean package'
        }

    }
  
    stage('deploy') {
        steps{
            script{
                def resourceGroup = 'SJ-reource'
                def webAppName = 'sj-javaapp'
                withCredentials([azureServicePrincipal(credentialsId: "${AZURE_CREDENTIALS_ID}",
                        subscriptionIdVariable: "${AZURE_SUBSCRIPTION_ID}",
                        clientIdVariable: "${AZURE_CLIENT_ID}",
                        clientSecretVariable: "${AZURE_CLIENT_SECRET}",
                        tenantIdVariable: "${AZURE_TENANT_ID}")]) 

              {
                    sh """
                    az login --service-principal --username \${AZURE_CLIENT_ID} --password \${AZURE_CLIENT_SECRET} --tenant \${AZURE_TENANT_ID}
                    az account set -s \${AZURE_SUBSCRIPTION_ID}
         
                    """
                }
                      // {
                      //      sh "az account set -s ${AZURE_SUBSCRIPTION_ID}"
                      //  }

                 // withCredentials([usernamePassword(credentialsId: "${AZURE_CREDENTIALS_ID}", passwordVariable: '0j58Q~IUMqGzuFHQOb-nIXQQ26vmpr_iNL26aa~-', usernameVariable: '46a1959c-4ab6-44c1-a70a-6a6aeadfad85')]) {
                 // sh " az login --service-principal -u ${AZURE_CLIENT_ID} -p ${AZURE_CLIENT_SECRET} -t ${AZURE_TENANT_ID} "
                    
                 // sh " az account set -s ${AZURE_SUBSCRIPTION_ID} "    
                    
                 // }
                // // get publish settings
                def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
                def ftpProfile = getFtpPublishProfile pubProfilesJson
                // upload package
                sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
                // log out
                sh 'az logout'
            }
        }
      
    }
  }
}
