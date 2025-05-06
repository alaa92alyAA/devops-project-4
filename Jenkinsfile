pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }

    environment {
        SNAP_REPO      = 'nexus-snapshot'
        NEXUS_USER     = 'admin'
        NEXUS_PASS     = 'P@ss1234'
        RELEASE_REPO   = 'nexus-repo'
        CENTRAL_REPO   = 'maven-central'
        NEXUSIP        = '10.0.156.185'
        NEXUSPORT      = '8081'
        NEXUS_GRP_REPO = 'nexus-group'
        NEXUS_LOGIN    = 'nexuslogin'
    }
}

stages {
    stage('Build') {
        steps {
            sh 'mvn -s settings.xml -DskipTests install'
        }
    }
}

