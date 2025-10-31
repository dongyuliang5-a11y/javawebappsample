import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=45136207-d203-4880-abc0-cf8cc44756f8',
        'AZURE_TENANT_ID=ce17aa49-d1e4-4135-b453-dd0b891e8b61']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName   = 'yourname-jenkins-demo'    // 换成你的 App 名
      def warFile      = 'target/calculator-1.0.war'// 打包后实际 WAR 名，按需改

      withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
    sh '''
      az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
      az account set -s $AZURE_SUBSCRIPTION_ID
    '''
  }
     // (1) 设定 Linux 运行时栈（幂等，多次执行也没问题）
  sh "az webapp config set -g ${resourceGroup} -n ${webAppName} --linux-fx-version 'TOMCAT|9.0-java11'"

  // (2) 部署 WAR
  //    A. 如果希望以根路径访问，直接把目标设为 ROOT.war（推荐）
  sh "az webapp deploy -g ${resourceGroup} -n ${webAppName} --src-path ${warFile} --type war --target-path webapps/ROOT.war"

  //    B. 如果保留原 WAR 名，上下文路径会是 /calculator-1.0（可选）：
  // sh "az webapp deploy -g ${resourceGroup} -n ${webAppName} --src-path ${warFile} --type war"

  // (3) 重启应用
  sh "az webapp restart -g ${resourceGroup} -n ${webAppName}"

  // 登出
  sh 'az logout'
}
