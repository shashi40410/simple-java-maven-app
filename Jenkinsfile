pipeline {
    agent { label 'MySlave1' }
    tools {
        maven 'Maven_3.9.6'
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
    }
}

