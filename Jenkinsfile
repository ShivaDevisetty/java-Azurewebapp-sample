import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=bdc39b31-8eb9-427b-8e9f-4684ff2ceb92',
        'AZURE_TENANT_ID=616ae545-5db6-48ed-9651-732703b94552']) {
    stage('init') {
      checkout scm
    }
   
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'Jenkins-Resourcegroup'
      def webAppName = 'Jenkins-Webapp'
      // login Azure
 //     withCredentials([usernamePassword(credentialsId: 'siva', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
       sh '''
          az login --service-principal -u tH38Q~oTjPhB-x2vJvyjWzas5xzSw2nt2DeuKbLx -p e8877f32-cfee-43d0-868b-10c3bbfc7b5a -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
    //  }
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
