import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=04e6c63f-c520-4526-97ef-1aeb15247ba1',
        'AZURE_TENANT_ID=621117aa-b444-437a-ac66-bc83a3c2f0be']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'nishu'
      def webAppName = 'ballbat'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'nishu', passwordVariable: 'EYl8Q~-ABv~oKVBm3ehbISrxGNJ9zbbklkg3EbA4', usernameVariable: '7e9a106c-6ab5-4c27-b45b-28e174f9ad57')]) {
       sh '''
          az login --service-principal -u 7e9a106c-6ab5-4c27-b45b-28e174f9ad57 -p EYl8Q~-ABv~oKVBm3ehbISrxGNJ9zbbklkg3EbA4 -t 621117aa-b444-437a-ac66-bc83a3c2f0be
          az account set -s 04e6c63f-c520-4526-97ef-1aeb15247ba1
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
