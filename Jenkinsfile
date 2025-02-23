pipeline{
    agent any
    tools {
        jdk "JDK17"
        maven "MAVEN3.9"
    }
    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS =  'admin'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUSIP = '172.31.22.101'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexuslogin'       
    }

    stages {
        stage('Build'){
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                         echo "Now Archiving."
                         archiveArtifact artifact: '**/*.war'
        }
        }
    }
    stage('Test'){
        steps {
            sh 'mvn -s settings.xml test'
        }
    }

    stage('Checkstyle Analysis'){
        steps{
            sh 'mvn -s settings.xml checkstyle:checkstyle'
        }
    }
    }
}