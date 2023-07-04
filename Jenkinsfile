import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles) {
    if (p['publishMethod'] == 'FTP') {
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
    }
  }
  return null
}

node {
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
    withCredentials([azureServicePrincipal(credentialsId: 'da863f92-0a7e-4198-b7bb-be20e7b9dbca', variable: 'AZURE_CREDENTIALS')]) {
      sh '''
        az login --service-principal -u d2beb018-8f93-4e2c-bfd4-bbf9535e4ef8 -p J_S8Q~JNF7K-Bb-i6VPUcu1mshDiheJmA-em3ae6 --tenant 507fc407-e0b7-4432-be14-ffb107fb7e24
      '''
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName --query '[].{publishMethod:publishMethod, publishUrl:publishUrl, userName:userName, userPWD:userPWD}' -o json", returnStdout: true
      def ftpProfile = getFtpPublishProfile(pubProfilesJson)
      if (ftpProfile) {
        // upload package
        sh "curl -T target/calculator-1.0.war ${ftpProfile.url}/webapps/ROOT.war -u '${ftpProfile.username}:${ftpProfile.password}'"
      } else {
        error 'FTP publish profile not found'
      }
      // log out
      sh 'az logout'
    }
  }
}
