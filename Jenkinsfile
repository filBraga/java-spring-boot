pipeline {
  agent any
  environment {
    TOMCAT_CREDS = credentials('ssh-key')
    TOMCAT_SERVER = "3.145.86.11"
    ROOT_WAR_LOCATION = "/opt/apache-tomcat-9.0.98/webapps"
    LOCAL_WAR_DIR = "/var/lib/jenkins/workspace/build-deploy-java/target/"
    WAR_FILE = "Filipe-0.0.1-SNAPSHOT.war"
  }
  stages {
    stage('GetProject') {
      steps {
        git branch: 'main', url: 'https://github.com/SimonCuddihy/simonspetitions.git'
      }
    }
    stage('Build') {
      steps {
        sh 'mvn clean compile'
      }
    }
    stage('Test') {
      steps {
        sh 'mvn test'
      }
    }
    stage('Package') {
      steps {
        sh 'mvn package'
      }
    }
    stage('Archive') {
      steps {
        archiveArtifacts allowEmptyArchive: true, artifacts: '**/target/*.war'
      }
    }
    stage('Approve Deployment') {
      steps {
        input message: 'Deploy to production?', ok: 'Deploy'
      }
    }
    stage('copy the war file to the Tomcat server') {
      steps {
        sh '''
          ssh -i $TOMCAT_CREDS $TOMCAT_CREDS_USR@$TOMCAT_SERVER "/opt/apache-tomcat-9.0.98/bin/catalina.sh stop"
          ssh -i $TOMCAT_CREDS $TOMCAT_CREDS_USR@$TOMCAT_SERVER "rm -rf $ROOT_WAR_LOCATION/ROOT; rm -f $ROOT_WAR_LOCATION/ROOT.war"
          scp -i $TOMCAT_CREDS $LOCAL_WAR_DIR/$WAR_FILE $TOMCAT_CREDS_USR@$TOMCAT_SERVER:$ROOT_WAR_LOCATION/ROOT.war
          ssh -i $TOMCAT_CREDS $TOMCAT_CREDS_USR@$TOMCAT_SERVER "chown $TOMCAT_CREDS_USR:$TOMCAT_CREDS_USR $ROOT_WAR_LOCATION/ROOT.war"
          ssh -i $TOMCAT_CREDS $TOMCAT_CREDS_USR@$TOMCAT_SERVER "/opt/apache-tomcat-9.0.98/bin/catalina.sh start"
        '''
      }
    }
  }
}