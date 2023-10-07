pipeline {
    agent any
    tools {
        maven "MAVEN"
        jdk "JAVA"
    }
    stages {
         stage('Fetch Code') {
            steps {
                git branch: 'vp-rem', url: 'https://github.com/satishnare/vprofile-project.git'
            }
         }

         stage('BUILD'){
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
         stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }
         stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }
         stage('CODE ANALYSIS with SONARQUBE') {

            environment {
                scannerHome = tool 'SONAR'
            }

            steps {
                withSonarQubeEnv('SONAR') {
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

         stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
         stage("UploadArtifact"){
            steps{
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: '172.31.1.5:8081',
                  groupId: 'QA',
                  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                  repository: 'vprofile-repo',
                  credentialsId: 'nexuslogin',
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
        success {
            emailext body: 'Your project has been built, tested, and deployed successfully!',
                     recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                     subject: 'V-Profile Deployment Notification',
                     to: 'satish.nare@yahoo.com'
        }
        failure {
            emailext body: 'Your project has failed to build, test, or deploy!',
                     recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                     subject: 'V-Profile Deployment Notification',
                     to: 'satish.nare@yahoo.com'
        }

}

}
        


           

