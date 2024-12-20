pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        SONAR_HOST_URL = 'http://3.109.186.241:9000' // SonarQube server URL
        SONAR_PROJECT_KEY = 'org.springframework:gs-maven'
        SONAR_PROJECT_NAME = 'gs-maven'
        NEXUS_URL = 'http://65.2.143.128:8081/repository/maven-snapshots/' // Nexus Snapshot Repository URL
        TOMCAT_HOST = 'http://65.0.168.203:8080'
        TOMCAT_USER = 'admin'
        TOMCAT_PASSWORD = 'Sushmi@2001'
        TOMCAT_DEPLOY_URL = "http://${TOMCAT_USER}:${TOMCAT_PASSWORD}@${TOMCAT_HOST}:8080/manager/text/deploy?path=/gs-maven&update=true"
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
                        // Manually extracting details from pom.xml
                        def pomFile = readFile('pom.xml')
                        def groupId = pomFile =~ /<groupId>(.*?)<\/groupId>/[0][1]
                        def artifactId = pomFile =~ /<artifactId>(.*?)<\/artifactId>/[0][1]
                        def version = pomFile =~ /<version>(.*?)<\/version>/[0][1]
                        def packaging = pomFile =~ /<packaging>(.*?)<\/packaging>/[0][1] ?: 'jar' // Default to 'jar' if packaging is missing

                        // Using credentials for Nexus upload
                        withCredentials([usernamePassword(credentialsId: 'nexus_credentials', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASSWORD')]) {
                            sh """
                                mvn deploy:deploy-file \
                                    -Durl=${NEXUS_URL} \
                                    -DrepositoryId=maven-snapshots \
                                    -Dfile=target/${artifactId}-${version}.${packaging} \
                                    -DgroupId=${groupId} \
                                    -DartifactId=${artifactId} \
                                    -Dversion=${version} \
                                    -Dpackaging=${packaging} \
                                    -Dusername=${NEXUS_USER} \
                                    -Dpassword=${NEXUS_PASSWORD}
                            """
                        }
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
