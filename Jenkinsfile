pipeline {
    agent any

    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }

    environment {
        NEXUS_IP       = "172.31.63.4"
        NEXUS_PORT     = "8081"
        NEXUS_USER     = "admin"
        NEXUS_PASS     = "NexusInstance"
        NEXUS_LOGIN    = "nexuslogin"
        NEXUS_GRP_REPO = "vpro-maven-group"
        RELEASE_REPO   = "vprofile-release"
        CENTRAL_REPO   = "vpro-maven-central"
        SNAP_REPO      = "vprofile-snapshot"
        SONAR_INSTANCE  = "sonarinstance"
        SONAR_SCANNER   = "sonarscanner"
    }

    stages {
        stage("Build") {
            steps {
                sh "mvn -s settings.xml -DskipTests install"
            }
            post {
                success {
                    echo "Success! Now Archiving"
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage("Test") {
            steps {
                sh "mvn test"
            }
        }
        stage("Checkstyle Analysis") {
            steps {
                sh "mvn checkstyle:checkstyle"
            }
        }
        stage("Sonar Analysis") {
            environment {
                scanner = tool "${SONAR_SCANNER}"
            }
            steps {
                withSonarQubeEnv("${SONAR_INSTANCE}") {
                    sh '''${scanner}/bin/sonar-scanner \
                        -Dsonar.projectKey=cicd-jenkins \
                        -Dsonar.projectName=cicd-jenkins \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                        '''
                }
            }
        }
        stage("Sonar Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage("Upload Artifact to Nexus") {
            steps {
                nexusArtifactUploader(
                    nexusVersion  : "nexus3",
                    protocol      : "http",
                    nexusUrl      : "${NEXUS_IP}:${NEXUS_PORT}",
                    groupId       : "cicdproject",
                    version       : "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository    : "${RELEASE_REPO}",
                    credentialsId : "${NEXUS_LOGIN}",
                    artifacts: [
                        [
                            artifactId: "java_app",
                            classifier: "",
                            file      : "target/vprofile-v2.war",
                            type      : "war"
                        ]
                    ]
                )
            }
        }

    }
}