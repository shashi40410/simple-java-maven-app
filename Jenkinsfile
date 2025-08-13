pipeline {
    agent { label 'MySlave1' }

    tools {
        maven 'Maven_3.9.6'
        // Using system-installed Java 17, so no jdk declaration needed
    }

    environment {
        SONARQUBE_ENV = 'MySonarQube'           // Jenkins SonarQube server name
        SONAR_PROJECT_KEY = 'jenkins-sonar-demo'
        SONAR_PROJECT_NAME = 'Jenkins Sonar Demo'
        SONAR_HOST_URL = 'http://192.168.56.21:9000'
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Cloning GitHub repo..."
                git branch: 'main', url: 'https://github.com/shashi40410/simple-java-maven-app.git'
            }
        }

        stage('Build') {
            steps {
                echo "Building project..."
                sh 'mvn -B -DskipTests clean package'
            }
        }

        stage('Test') { 
            steps {
                echo "Running unit tests..."
                sh 'mvn test'
            }
            post {
                always {
                    echo "Publishing test results..."
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "Running SonarQube analysis..."
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    withCredentials([string(credentialsId: 'jenkins-sonar-demo', variable: 'SONAR_TOKEN')]) {
                        sh "mvn sonar:sonar \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.projectName='${SONAR_PROJECT_NAME}' \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=${SONAR_TOKEN}"
                    }
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
