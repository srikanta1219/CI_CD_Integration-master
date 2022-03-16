def mvnHome

node('slave_docker'){
   stage('git checkout'){
      try {
      git credentialsId: 'githubCreds', url: 'https://github.com/srikanta1219/CI_CD_Integration-master.git'
      } catch(err) {
         sh "echo error in checkout"
      }
   }
  
   stage('maven test'){
      try {
      mvnHome=tool 'maven-3.6.3'
      sh "${mvnHome}/bin/mvn --version"
      sh "${mvnHome}/bin/mvn clean test surefire-report:report"
      } catch(err) {
         sh "echo error in defining maven"
      }
   }
   
   stage('test case and report'){
      try {
         echo "executing test cases"
         junit allowEmptyResults: true, testResults: 'addressbook_main/target/surefire-reports/*.xml'
         publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'addressbook_main/target/site', reportFiles: 'surefire-report.html', reportName: 'SureFireReportHTML', reportTitles: ''])
      } catch(err) {
         throw err
      }
   }
   
      stage('package and artifacts'){
      try {
         sh "${mvnHome}/bin/mvn clean package -DskipTests=true"
         archiveArtifacts allowEmptyArchive: true, artifacts: 'addressbook_main/target/**/*.war'
      } catch(err) {
         sh "echo error in generating artifacts"
      }
   }

   stage ('docker build and push'){
      try {
       sh "docker version"
       sh "docker build -t srikanta1219/archiveartifacts:newtag -f Dockerfile ."
       sh "docker run -p 8098:8080 -d srikanta1219/archiveartifacts:newtag"
       withDockerRegistry(credentialsId: 'DockerHub') {
       sh "docker push srikanta1219/archiveartifacts:newtag"
        }
      } catch(err) {
         sh "echo error in docker build and pushing to docker hub"
      }
   }

   stage('deployment of application') {
      try {
        sshagent(['ec2-user-target']){
           // clone the repo on target in tmp
            sh "ssh -o StrictHostKeyChecking=no ec2-user@54.234.138.217 /tmp/CI_CD_Integration/tomcat.sh"
            sh "scp -o StrictHostKeyChecking=no addressbook_main/target/addressbook.war ec2-user@54.234.138.217:/tmp"
            sh "ssh -o StrictHostKeyChecking=no ec2-user@54.234.138.217 /tmp/CI_CD_Integration/symlink_target.sh"
            }
        } catch(err) {
           sh "echo error in deployment of an application"
        }
   }
      
   stage('artifacts to s3') {
      try {
      // you need cloudbees aws credentials
      withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'deploytos3', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
         sh "aws s3 ls"
         sh "aws s3 cp addressbook_main/target/addressbook.war s3://cloudyeticicd/"
         }
      } catch(err) {
         sh "echo error in sending artifacts to s3"
      }
   }
}
