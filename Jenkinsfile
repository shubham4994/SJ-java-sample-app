import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=ec940544-e7ec-4f93-ade7-8e607a0b020b',
        'AZURE_TENANT_ID=507fc407-e0b7-4432-be14-ffb107fb7e24']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'SJ-reource'
      def webAppName = 'sj-javaapp'
      // login Azure
      withCredentials([usernamePassword(credentialsId: '848ac45f-f8e2-4d34-b119-9e1138ddaaf6', passwordVariable: 'J_S8Q~JNF7K-Bb-i6VPUcu1mshDiheJmA-em3ae6', usernameVariable: 'd2beb018-8f93-4e2c-bfd4-bbf9535e4ef8')]) {
       sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}
