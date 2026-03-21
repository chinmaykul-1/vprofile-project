def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

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
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Publish to Nexus') {
            steps {
                    nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                    groupId: 'QA',
                    version: "${env.BUILD_NUMBER}-${env.BUILD_TIMESTAMP}",
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


    }

post {
    always {
        script {
            echo 'Sending build status to Slack...'
            slackSend channel: '#jenkinscicd', color: COLOR_MAP[currentBuild.currentResult], message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} \n More info at : ${env.BUILD_NUMBER} ${env.BUILD_URL}"
        }
    }
}
}