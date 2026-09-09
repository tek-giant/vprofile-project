pipeline {
    agent any

    tools {
        maven "MAVEN3.9.9"
        jdk "JDK17"
    }

    environment {
        SNAP_REPO      = 'vprofile-snapshot'
        NEXUS_USER     = 'admin'
        NEXUS_PASS     = 'admin123'
        RELEASE_REPO   = 'vprofile-release'
        CENTRAL_REPO   = 'vpro-maven-central'
        NEXUSIP        = '172.31.95.106'
        NEXUSPORT      = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN    = 'nexuslogin'
        SONARSERVER    = 'sonarserver'
        SONARSCANNER   = 'sonarscanner'
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn -U -s settings.xml -DskipTests install'
            }

            post {
                success {
                    echo 'Now Archiving.'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('Test') {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            steps {
                script {
                    def scannerHome = tool env.SONARSCANNER
                    def sonarJavaHome = tool 'JDK11'

                    withSonarQubeEnv(env.SONARSERVER) {
                        withEnv([
                            "JAVA_HOME=${sonarJavaHome}",
                            "PATH+SONARJAVA=${sonarJavaHome}/bin",
                            "SCANNER_HOME=${scannerHome}"
                        ]) {
                            sh '''
                                java -version

                                $SCANNER_HOME/bin/sonar-scanner \
                                  -Dsonar.projectKey=vprofile \
                                  -Dsonar.projectName=vprofile \
                                  -Dsonar.projectVersion=1.0 \
                                  -Dsonar.sources=src/main \
                                  -Dsonar.tests=src/test \
                                  -Dsonar.java.binaries=target/classes \
                                  -Dsonar.junit.reportPaths=target/surefire-reports \
                                  -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml \
                                  -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                            '''
                        }
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Upload Artifact') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${env.NEXUSIP}:${env.NEXUSPORT}",
                    groupId: 'QA',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: env.RELEASE_REPO,
                    credentialsId: env.NEXUS_LOGIN,
                    artifacts: [
                        [
                            artifactId: 'vproapp',
                            classifier: '',
                            file: 'target/vprofile-v2.war',
                            type: 'war'
                        ]
                    ]
                )
            }
        }
    }
}