pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        SONAR_HOST_URL = 'http://3.109.186.241:9000' // SonarQube server URL
        SONAR_PROJECT_KEY = 'org.springframework:gs-maven'
        SONAR_PROJECT_NAME = 'gs-maven'
        NEXUS_URL = 'http://65.2.143.128:8081/' // Nexus HTTP URL
        NEXUS_CREDENTIALS = 'nexus_credentials' // ID for Nexus credentials
        TOMCAT_HOST = 'http://65.0.168.203:8080'
        TOMCAT_USER = 'admin'
        TOMCAT_PASSWORD = 'Sushmi@2001'
        TOMCAT_DEPLOY_URL = "http://${TOMCAT_USER}:${TOMCAT_PASSWORD}@${TOMCAT_HOST}/manager/text/deploy?path=/gs-maven&update=true"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', credentialsId: 'git_hub', url: 'https://github.com/Rama3058/gs-maven.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    dir('complete') {
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
            }
        }

        stage('Build') {
            steps {
                script {
                    dir('complete') {
                        sh 'mvn clean package'
                    }
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                script {
                    dir('complete') {
                        withCredentials([usernamePassword(credentialsId: NEXUS_CREDENTIALS, usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                            writeFile file: 'settings.xml', text: """
<settings>
    <servers>
        <server>
            <id>nexus-releases</id>
            <username>${NEXUS_USER}</username>
            <password>${NEXUS_PASS}</password>
        </server>
    </servers>
</settings>
"""
                            sh """
                                mvn deploy --settings settings.xml \
                                    -DaltDeploymentRepository=nexus-releases::default::${NEXUS_URL}
                            """
                        }
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    dir('complete') {
                        sh """
                            curl -T target/gs-maven.war "${TOMCAT_DEPLOY_URL}"
                        """
                    }
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
