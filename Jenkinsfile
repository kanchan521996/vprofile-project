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
                    archiveArtifacts '**/*.war'
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
                    '''
                }
            }
        }

        stage('Upload artifact') {
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
                        [artifactId: 'vproapp',
                         classifier: '',
                         file: 'target/vprofile-v2.war',
                         type: 'war']
                    ]
                )
            }
        }

  stage('Pull Artifact from Nexus & Push to GitHub') {
    steps {
        script {
            // Dynamically generate artifact version (replace colons and spaces with dashes)
            def artifact_version = "17-25-02-23_08-47" // Replace with dynamic value if needed
            def artifact_name = "vproapp-${artifact_version}.war"
            def nexus_url = "http://${NEXUSIP}:${NEXUSPORT}/repository/${RELEASE_REPO}/QA/vproapp/${artifact_version}/${artifact_name}"
            def branch = "testing"

            // Debugging: Print Nexus URL and credentials
            echo "Generated Nexus URL: ${nexus_url}"
            echo "Nexus User: ${NEXUS_USER}"
            echo "Nexus Password: ${NEXUS_PASS}"

            // 1️⃣ Download the artifact from Nexus
            sh """
                wget --user=${NEXUS_USER} --password=${NEXUS_PASS} \
                "${nexus_url}" \
                -O ${artifact_name}
            """

            // 2️⃣ Setup Git & Push to GitHub Testing Branch
            sh """
                git config --global user.email "kanchannath819@gmail.com"
                git config --global user.name "nath"
                git checkout ${branch}
                git pull origin ${branch}
                mv ${artifact_name} .
                git add ${artifact_name}
                git commit -m "Pushed Nexus artifact ${artifact_name} to GitHub testing branch"
                git push origin ${branch}
            """
        }
    }
}
    }
}