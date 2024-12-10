pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        SONAR_HOST_URL = 'http://3.109.186.241:9000'
        SONAR_PROJECT_KEY = 'org.springframework:gs-maven'
        SONAR_PROJECT_NAME = 'gs-maven'
        NEXUS_URL = 'http://65.2.143.128:8081/repository/demo-snapshots'
        NEXUS_CREDENTIALS = 'nexus_credentials'
        TOMCAT_HOST = 'http://65.0.168.203:8080'
        TOMCAT_USER = 'admin'
        TOMCAT_PASSWORD = 'Sushmi@2001'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', credentialsId: 'git_hub', url: 'https://github.com/Rama3058/gs-maven.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'SONAR_TOKEN', usernameVariable: 'SONAR_USER', passwordVariable: 'SONAR_TOKEN')]) {
                    sh """
                        mvn clean verify sonar:sonar \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Upload to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: NEXUS_CREDENTIALS, usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    writeFile file: 'settings.xml', text: """
<settings>
    <servers>
        <server>
            <id>nexus-snapshots</id>
            <username>${NEXUS_USER}</username>
            <password>${NEXUS_PASS}</password>
        </server>
    </servers>
</settings>
"""
                    sh "mvn deploy --settings settings.xml"
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
