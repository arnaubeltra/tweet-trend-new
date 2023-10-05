def registry = 'https://arnaubeltragft.jfrog.io'            // Artifactory repo url

pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    // As mvn binary is available at /opt/apache-maven-3.9.4/bin, we add this path to the PATH so that we can run 'mvn' at the stage.
    environment {
        PATH = "/opt/apache-maven-3.9.4/bin:$PATH"
    }

    stages {
        stage("build") {
            steps {
                echo "------------ Build started ------------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "----------- Build completed -----------"
            }
        }

        stage("test") {
            steps {
                echo "---------- Unit test started ----------"
                sh 'mvn surefire-report:report'
                echo "--------- Unit test completed ---------"
            }
        }
    
        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'valaxy-sonar-scanner';              // 'valaxy-sonar-scanner' is the scanner name configured at Manage Jenkins > Tools > SonarQube Scanner Name. Used to set the scanner and version.
            }

            steps {
                withSonarQubeEnv('valaxy-sonarqube-server') {           // 'valaxy-sonarqube-server' is the server name configured at Manage Jenkins > System > SonarQube servers.
                    sh "${scannerHome}/bin/sonar-scanner"               // Comunicates with SonarQube server and send the analysis report.
                }
            }
        }

        // Quality gate is used to sync the final status of the Jenkins pipeline with the status of the Sonar scan. If Sonar scan fails, Quality gate is set to error so Jenkins pipeline fails.
        stage("Quality Gate") {                                         
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {                   // Just in case something goes wrong, pipeline will be killed after a timeout
                        def qg = waitForQualityGate()                   // Reuse taskId previously collected by withSonarQubeEnv
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage("Publish Jar to Artifactory") {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'
                    def server = Artifactory.newServer url:registry + "/artifactory" ,  credentialsId: "artifactory-cred"           // Configure the Artifactory repo url (https://arnaubeltragft.jfrog.io/artifactory)
                    def properties = "buildid = ${env.BUILD_ID}, commitid = ${GIT_COMMIT}";
                    // our jar file is stored at the worker at ~/jenkins/workspace/_trend_multibranch_pipeline_main/jarstaging/com/valaxy/demo-workshop/2.1.2 (the path is set according to what has been defined at the pom.xml)
                    // it will look for all files (except .sha1 and .md5 files) inside the jarstaging folder (so it is taking the jar and a pom), and it will store them at the Artifactory repo at libs-release-local folder.
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/(*)",
                                "target": "libs-release-local/{1}",
                                "flat": "false",
                                "props" : "${properties}",
                                "exclusions": [ "*.sha1", "*.md5"]
                            }
                        ]
                    }"""
                    def buildInfo = server.upload(uploadSpec)               // uploads the artifacts to the server
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)
                    echo '<--------------- Jar Publish Ended --------------->'
                }
            }   
        } 
    }
}