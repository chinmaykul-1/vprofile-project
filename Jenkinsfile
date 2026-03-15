pipeline {
    agent any
    tools {
        maven "MAVEN3.0"
        jdk "JDK17"
    }
    
    environment {
        SNAP_REPO = 'vprofile-snapshot'
		NEXUS_USER = 'admin'
		NEXUS_PASS = 'adminpass'
		RELEASE_REPO = 'vprofile-release'
		CENTRAL_REPO = 'vpro-maven-central'
		NEXUSIP = '172.31.10.42'
		NEXUSPORT = '8081'
		NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexuscreds'
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
    }

    stages {
        stage('Build'){
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo 'Build successful, now archiving the artifact...'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Test'){
            steps {
                sh 'mvn -s settings.xml test'
            }
        }
        stage('Checkstyle analysis'){
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }
        stage('SonarQube analysis'){
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
                withSonarQubeEnv("${SONARSERVER}") {
                    sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile -Dsonar.projectName=vprofile  -Dsonar.projectVersion=1.0  -Dsonar.sources=src/  -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest -Dsonar.junit.reportsPath=target/surefire-reports/ -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml -Dsonar.jacoco.reportPaths=target/jacoco.exec"
                }
            }
        }
    }
}