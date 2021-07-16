import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=1667381d-56e2-45f5-ac37-0870394e4035',
        'AZURE_TENANT_ID=c61dbbd0-0f96-4a10-882e-e178f1a6e278',
           'AZURE_CLIENT_SECRET=SroN3CmER4ZFB_4LHKcg4b45Gl86Sxmk__',
           'AZURE_CLIENT_ID=88a1e8ef-e386-4b1f-b6a0-f4c9706fd40d',
           'AZURE_TENANT_ID=c61dbbd0-0f96-4a10-882e-e178f1a6e278'
          
          ]) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'QuickstartJenkins-rg'
      def webAppName = 'ejaleeJenkinsApp'
      
      // login Azure
      withCredentials([usernamePassword(credentialsId: '60896e1b-0dec-4160-be4b-a4e45d9f13e4', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
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
