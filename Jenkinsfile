/* pipeline {
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
         stage('Test') { 
            steps {
                sh 'mvn test' 
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml' 
                }
            }
        }
         stage('Deliver') { 
            steps {
                sh './jenkins/scripts/deliver.sh' 
            }
        }
    }
}  
*/

pipeline {
    agent { label 'MySlave1' }
    tools {
        maven 'Maven_3.9.6'
    }
    environment {
        // This should match the name configured in Jenkins > Manage Jenkins > Configure System > SonarQube servers
        SONARQUBE_ENV = 'MySonarQube'
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }

        stage('Test') { 
            steps {
                sh 'mvn test' 
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml' 
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Wrap inside SonarQube environment
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=my-project-key -Dsonar.host.url=172.28.130.32:9000 -Dsonar.login=1190afc7e9d7e2e912031e663687ebe7c5'
                }
            }
        }

        stage('Deliver') { 
            steps {
                sh './jenkins/scripts/deliver.sh' 
            }
        }

        stage('Quality Gate') {
            steps {
                // This waits for SonarQube quality gate result before continuing
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
