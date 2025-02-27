pipeline{
    agent any

    environment{
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "172.31.27.67:8081"
        NEXUS_REPOSITORY = "vprofile-repo"
        NEXUS_CREDENTIAL_ID = "nexus-cred" 
        ARTVERSION = "${env.BUILD_ID}"
        scannerHome = tool 'Sonar-4.7.0.2747'
        NEXUS_REPOGRP_ID = "vprofile-grp-repo"
        
    }
    stages{
        stage('Fetch Code'){
            steps{
                git branch: 'paac', url:'https://github.com/chia5/vprofile-project.git'
            }
        }
        stage('Build') {
            steps{
                sh 'mvn clean install -DskipTests'
            }
            post{
                success{
                    echo "Now Archiving Artifact"
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }
        stage('Unit Test') {
            steps{
                sh 'mvn test'
            }
        } 
        stage('Intergration Test'){
            steps{
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage('Code Analysis with Checkstyle'){
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
            post{
                success{
                    echo "Generated Code Analysis Report"
                }
            }
        }

        stage('Code Analysis with Sonarqube'){
            environment{
                scannerHome = tool 'Sonar-4.7.0.2747'
            }
            steps{
                withSonarQubeEnv('Sonar-vprofile'){
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
                timeout(time:10, unit:'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version} ARTVERSION";
                        nexusArtifactUploader(
                                nexusVersion: NEXUS_VERSION,
                                protocol: NEXUS_PROTOCOL,
                                nexusUrl: NEXUS_URL,
                                groupId: pom.groupId,
                                version: ARTVERSION,
                                repository: NEXUS_REPOSITORY,
                                credentialsId: NEXUS_CREDENTIAL_ID,
                                artifacts: [
                                        [artifactId: pom.artifactId,
                                         classifier: '',
                                         file: artifactPath,
                                         type: pom.packaging],
                                        [artifactId: pom.artifactId,
                                         classifier: '',
                                         file: "pom.xml",
                                         type: "pom"]
                                ]
                        );
                    }
                    else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }

    }
}
