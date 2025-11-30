pipeline {
    agent any
    tools {
        jdk "JDK17"
        maven "MAVEN3.9"
    }
    environment {
        
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUSIP = '98.93.17.167'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
        SONARSERVER = 'sonarserver'
    }
    stages {
        stage ("Build"){
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo "Now archiveing"
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage ('Test'){
            steps {
                sh 'mvn -s settings.xml test'
            }
        }
        stage ('Checkstyle analysis'){
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }
        stage ('Artifact Upload'){
            steps {
                   nexusArtifactUploader(
        nexusVersion: 'nexus3',
        protocol: 'http',
        nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
        groupId: 'QA',
        version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
        repository: "${RELEASE_REPO}",
        credentialsId: "${NEXUS_LOGIN}",
        artifacts: [
            [artifactId: 'vprofile',
             classifier: '',
             file: 'target/vprofile-v2.war',
             type: 'war']
        ]
     )
     
        }
        }

    }
}