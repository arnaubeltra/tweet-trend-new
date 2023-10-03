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
    }
}