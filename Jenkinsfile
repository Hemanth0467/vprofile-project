pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }
    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUSIP = '172.31.89.33'
        NEXUSPORT = '8081'
        NEXUS_LOGIN = 'nexuslogin' 
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
    }
    
    stages {
        stage('Git-Checkout') {
            steps {
                git branch: 'ci-jenkins', url: 'https://github.com/Hemanth0467/vprofile-project.git'
            }
        }
        
        stage('Unit-Testing') {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }
        
        stage('Integration-Testing') {
            steps {
                sh 'mvn -s settings.xml verify -DskipUnitTests'
            }
        }
        
        
        stage('CODE ANALYSIS with SONARQUBE') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
                        }
            
            steps {
                withSonarQubeEnv("${SONARSERVER}") {
                  sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                               -Dsonar.projectName=vprofile-repo \
                               -Dsonar.projectVersion=1.0 \
                               -Dsonar.sources=src/ \
                               -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                               -Dsonar.junit.reportsPath=target/surefire-reports/ \
                               -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                               -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                        }
            
                        
                   }
        }
        
        stage('Quality-Gate-check') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Artifact') {
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        
        stage('Upload Artifact to Nexus') {
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
                        file: 'target/vprofile-4.3.war',
                        type: 'war']
                        ]
                    )
            }
        }
           
        
    }
}