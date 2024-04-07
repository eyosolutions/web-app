def COLOR_MAP = [
  'SUCCESS': 'good',
  'FAILURE': 'danger',
]

pipeline {
  agent any
  tools {
    maven "maven3.9.6"
    jdk "openJDK17"
  }
  stages {
    stage('Git Clone') {
      steps {
        git branch: 'main', url: 'https://github.com/eyosolutions/web-app.git'
        sh "echo end of git clone"
      }
    }
    stage('Maven build') {
      steps {
        sh 'mvn clean'
        sh "echo maven clean done."
      }
    }
    stage('Maven test') {
      steps {
        sh 'mvn test'
        sh "echo maven test done."
      }
    }
    stage('Maven package') {
      steps {
        sh 'mvn package'
        sh "echo maven package done."
      }
    }
    stage('Sonar Analysis') {
      environment {
        scannerHome = tool 'sonar5.0'
      }
      steps {
        withSonarQubeEnv('sonarqube') {
          sh '${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=jomacs'
        }
      }
    }
    stage('Quality Gate') {
      steps {
        timeout(time: 2, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
    stage('Upload Artifact') {
      steps {
        nexusArtifactUploader(
          nexusVersion: 'nexus3',
          protocol: 'http',
          nexusUrl: '172.31.24.16:8081',
          groupId: 'com.mt',
          version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}-RELEASE",
          repository: 'webapp-release',
          credentialsId: 'nexus-id',
          artifacts: [
              [artifactId: 'maven-web-application',
              classifier: '',
              file: 'target/web-app.war',
              type: 'war']
          ]
        )
      }
    }
    stage('Deploy to UAT') {
      steps {
        sh "echo start deploying to server in UAT Env"
        deploy adapters: [tomcat9(credentialsId: 'tomcat-id', path: '', url: 'http://172.31.38.154:8080')],
        contextPath: null,
        war: 'target/*.war'
      } 
    }

  }
  // After stages
  post {
    always {
      echo 'Slack Notifications.'
      slackSend channel: '#testing',
        color: COLOR_MAP[currentBuild.currentResult],
        message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
    }
  }
}


/*
pipeline{
  agent any
  tools{
    maven "maven3.8.8"
  }

    stages{
      stage("1. Git clone from repo"){
        steps{
         sh "echo start of git clone"
         git branch: 'main', url: 'https://github.com/JOMACS-IT/web-app.git'
         sh "echo end of git clone"
        }
      }
      
      stage("2. Build from Maven"){
        steps{
          sh "echo start building from Maven"
          sh "mvn clean package"
          sh "echo end of build"
        }  
      }
      
      stage("3. Code Scan"){
        steps{
          sh "echo start of code scan"
          sh "mvn sonar:sonar"
          sh "echo end of code scan"
        }  
      }
      
      stage("4. Store Artifacts"){
        steps{
           sh "echo Deploy Artifact"
           sh "mvn deploy"
        }  
      }
      
      stage("5. Deploying to Tomcat in UAT"){
        steps{
           sh "echo start deploying to server in UAT Env"
           deploy adapters: [tomcat9(credentialsId: 'tomcat_cred', path: '', url: 'http://18.117.162.68:9090')], contextPath: null, war: 'target/*.war'
        }  
      }
      
      stage("6. Email Notification"){
        steps{
            sh "echo Email Notification to DevOps Team"
            emailext body: 'The Deployment is Successful', subject: 'Deployment Success', to: 'info@jomacsit.com'
        }  
      }
    }
}
*/


/*
node{
    def MHD = tool name: "maven3.8.4"
    stage('code'){
        git branch: 'development', url: 'https://github.com/team16flight/web-app.git'
    }
    stage('BUILD'){
       sh "${MHD}/bin/mvn clean package"
 
    }
        
    
    stage('deploy'){
  sshagent(['tomcat']) {
  sh "scp -o StrictHostKeyChecking=no target/*war ec2-user@172.31.15.31:/opt/tomcat9/webapps/"
}
}
stage('email'){
emailext body: '''Build is over

JOMACS 
437212483''', recipientProviders: [developers(), requestor()], subject: 'Build', to: 'tdapp@gmail.com'
}
}
 */
