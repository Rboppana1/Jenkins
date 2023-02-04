pipeline {
    agent any

    stages {
        stage('Git Checkout'){
            steps {
                git branch: 'master', url: 'gitrmasterbranchurl'
            }

        }
        stage('UNIT Test'){
            steps{
                sh 'mvn test'
            }
        }
        stage('Maven Build'){
            steps{
                sh 'mvn clean install'
            }
        }
        stage('Static code analysis'){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-api'){
                     sh 'mvn clean package sonar:sonar'
                 }
                }
            }
        }
        stage('Quality Gate status'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-api'
                }
            }
        }
        stage('Upload file to Nexus'){
            steps{
                script{
                    def nexusRepo = readMavenPom.version.endsWith("SNAPSHOT") ? "integrationserviceapp-snapshot" : "integrationserviceapp-release"
                    nexusArtifactUploader artifacts:
                    [
                        [
                            artifactId: *****,
                            classifier: '', file: 'target/***.jar'
                            type: 'jar'
                            ]
                    ],
                    credentialsId: 'nexus-auth', groupId: 'com.example', nexusUrl: '*.***.***.**.*****', nexusVersion: 'nexus3',protocol: 'http', repository: nexusRepo, version: '*.*.*'
                }
            }
        }
        stage('APPROVAL'){
            steps{
                script{
                mail from: "$VM_EMAIL_FROM", to: "$VM_SUPPORT_EMAIL", subject: "APPROVAL REQUIRED FOR $JOB_NAME" , body: """Build $BUILD_NUMBER required an approval. Go to $BUILD_URL for more info."""
                }
            }
        }
        stage('Deploy to Tomcat'){
            steps{
                script{
                    curl "http://myrepo/nexus/****/****/artifact/maven/redirect?r=releases&g=myorg&a=myapp&v=*.*=war" \
                          -o /usr/local/share/tomcat7/webapps/*****app.war
                    service tomcat* restart
                }
            }
        }
        stage('Send email to dev team'){
            steps{
                script{
                    mail  body: 'STATUS OF BUILD', cc: 'developer', from: 'Jenkins-auto-generate', replyTo: '', subject: 'STATUS and Branch details', to: 'teamdl@gmail.com'
                }
            }
        }
        stage('post release tests on environment'){
            steps{
                //post release tests are done on environment defined.
                echo 'post release tests/write command for UAT tests here based on tool'
            }
        }
        stage('Send email to QA team'){
            steps{
                script{
                    mail  body: 'Test execution status', cc: 'qateam', from: 'Jenkins-auto-generate', subject: 'Test execution status', to: 'teamdl@outlook.com'
                }
            }
        }

    }
    post {
        always{
            //configure the path where we want the results to be published.
            junit 'build/reports/**/*.xml'
        }
    }
}