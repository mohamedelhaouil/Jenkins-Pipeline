pipeline {
    agent any

    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }

    environment {
        NEXUS_IP       = "172.31.63.4"
        NEXUS_PORT     = "8081"
        NEXUS_LOGIN    = "nexuslogin"
        NEXUS_USER     = "admin"
        NEXUS_PASS     = "admin123"
        NEXUS_GRP_REPO = "vpro-maven-group"
        RELEASE_REPO   = "vprofile-release"
        CENTRAL_REPO   = "vpro-maven-central"
        SNAP_REPO      = "vprofile-snapshot"
    }

    stages {
        stage("Build") {
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
        }
}