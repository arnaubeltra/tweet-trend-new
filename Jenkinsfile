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
                sh 'mvn clean deploy'
            }
        }
    
        stage('SonarQube analysis') {
            environment {
                // 'valaxy-sonar-scanner' is the scanner name configured at Manage Jenkins > Tools > SonarQube Scanner Name. Used to set the scanner and version.
                scannerHome = tool 'valaxy-sonar-scanner';
            }

            steps {
                // 'valaxy-sonarqube-server' is the server name configured at Manage Jenkins > System > SonarQube servers.
                withSonarQubeEnv('valaxy-sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner"           // Comunicates with SonarQube server and send the analysis report.
                }
            }
        }
    }
}