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
        NEXUSIP = '172.31.22.101'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexuslogin' 
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'    
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts '**/*.war'  // ✅ Corrected
                }
            }
        }

        stage('Test') {
            steps {
                sh 'mvn -s settings.xml test'
            }
            post {
                always {
                    echo "Test stage completed."
                }
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
            post {
                always {
                    echo "Checkstyle Analysis completed."
                }
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool SONARSCANNER
            }
            steps {
                withSonarQubeEnv(SONARSERVER) {
                    sh '''
                        export SONAR_SCANNER_OPTS="--add-opens java.base/java.lang=ALL-UNNAMED"
                        ${scannerHome}/bin/sonar-scanner -X \
                        -Dsonar.projectKey=vprofile \
                        -Dsonar.projectName=vprofile \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                    '''  // ✅ Fixed export issue and removed extra backslash
                }
            }
        }
        stage('Upload the artifact to nexus'){
            steps {
                sh '''
                 mvn -s settings.xml deploy \
    -DaltDeploymentRepository=${RELEASE_REPO}::default::http://${NEXUSIP}:${NEXUSPORT}/repository/${RELEASE_REPO}/ \
    -Dnexus.username=${NEXUS_USER} \
    -Dnexus.password=${NEXUS_PASS}
                '''
            }
            post {
                success {
                     echo "Artifact successfully deployed to Nexus."
                }
            }
        }
    }
}
