pipeline {
    agent { label 'MySlave1' }

    tools {
        maven 'Maven_3.9.6'
        // Optional: specify JDK if needed
      //  jdk 'JDK_17'
    }

    environment {
        // Jenkins SonarQube server name configured in Jenkins global configuration
        SONARQUBE_ENV = 'MySonarQube'

        // Optional: SonarQube project properties
        SONAR_HOST_URL = 'http://192.168.56.21:9000'
        SONAR_PROJECT_KEY = 'my-project-key'
        SONAR_PROJECT_NAME = 'My Project'
    }

    stages {
        stage('Build') {
            steps {
                echo "Building project..."
                sh 'mvn -B -DskipTests clean package'
            }
        }

        stage('Test') { 
            steps {
                echo "Running tests..."
                sh 'mvn test'
            }
            post {
                always {
                    echo "Publishing test reports..."
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "Running SonarQube analysis..."
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh "mvn sonar:sonar \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.projectName='${SONAR_PROJECT_NAME}' \
                        -Dsonar.host.url=${SONAR_HOST_URL}"
                }
            }
        }

        stage('Deliver') { 
            steps {
                echo "Running delivery script..."
                sh './jenkins/scripts/deliver.sh'
            }
        }

        stage('Quality Gate') {
            steps {
                echo "Waiting for SonarQube Quality Gate..."
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
