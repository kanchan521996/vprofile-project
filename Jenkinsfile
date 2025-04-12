pipeline {
    agent any
    tools {
        jdk "JDK17"
        maven "MAVEN3.9"
    }
    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUSIP = '34.207.60.219'
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
                    
                    artifacts: [
                        [artifactId: projectName,
                        classifier: '',
                        file: 'my-service-' + version + '.jar',
                        type: 'jar']
        ]
     )
     
        }
        }

    }
}