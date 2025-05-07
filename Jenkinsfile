pipeline {
    agent any

    /*
    tools {
        maven "maven3"
    }
    */

    environment {
        NEXUS_VERSION         = "nexus3"
        NEXUS_PROTOCOL        = "http"
        NEXUS_URL             = "10.0.156.185:8081"
        NEXUSIP               = "10.0.156.185"
        NEXUSPORT             = "8081"
        NEXUS_USER            = "admin"
        NEXUS_PASS            = "P@ss1234"
        SNAP_REPO             = "vprofile-snapshot"
        RELEASE_REPO          = "vprofile-release"
        CENTRAL_REPO          = "vpro-maven-central"     // fixed typo
        NEXUS_GRP_REPO        = "vpro-maven-group"
        NEXUS_LOGIN           = "nexuslogin"
        NEXUS_REPOSITORY      = "vprofile-release"
        NEXUS_REPO_ID         = "vprofile-release"
        NEXUS_CREDENTIAL_ID   = "nexuslogin"
        ARTVERSION            = "${env.BUILD_ID}"
    }

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Integration Test') {
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage('Code Analysis - Checkstyle') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('Code Analysis - SonarQube') {
            environment {
                scannerHome = tool 'sonarscanner4'
            }
            steps {
                withSonarQubeEnv('sonar-pro') {
                    sh '''${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=vprofile \
                        -Dsonar.projectName=vprofile-repo \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    def pom = readMavenPom file: "pom.xml"
                    def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    def artifactPath = filesByGlob[0].path
                    def artifactExists = fileExists artifactPath

                    echo "Artifact Info: ${filesByGlob[0].name}, ${artifactPath}"

                    if (artifactExists) {
                        echo "*** Uploading: ${artifactPath}, group: ${pom.groupId}, version: ${ARTVERSION}"

                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: ARTVERSION,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [
                                    artifactId: pom.artifactId,
                                    classifier: '',
                                    file: artifactPath,
                                    type: pom.packaging
                                ],
                                [
                                    artifactId: pom.artifactId,
                                    classifier: '',
                                    file: "pom.xml",
                                    type: "pom"
                                ]
                            ]
                        )
                    } else {
                        error "*** File: ${artifactPath} could not be found"
                    }
                }
            }
        }
    }
}
