pipeline {
    agent any
    tools {
        maven 'maven'
        ansible 'ansible'
    }
    stages {
        stage('Hello') {
            steps {
                git branch: 'main', url: 'https://github.com/vtricksshiva/java-cicd.git'
            }
        }

        stage('sonar_scan') {
            steps {
                script {
                    withSonarQubeEnv('Sonarqube') {
                        sh "mvn clean verify sonar:sonar -Dsonar.projectKey=sonar-scan-with-jenkins -Dsonar.projectName='sonar-scan-with-jenkins'"
                    }
                }
            }
        }

        stage('build binaries') {
            steps {
                sh 'mvn clean install'
                sh 'cp target/java-frontend-app.war .'
                sh 'mv java-frontend-app.war java-frontend-app-${BUILD_NUMBER}.war'
            }
        }

        stage('push binaries to nexus') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'nexus-cred', passwordVariable: 'pass', usernameVariable: 'user')]) {
                        sh """
                        curl -u ${user}:${pass} -T java-frontend-app-${BUILD_NUMBER}.war \
                        "http://54.91.74.205:8081/repository/java-app//java-frontend-app-${BUILD_NUMBER}.war"
                        """
                    }
                }
            }
        }

        stage('deploy to tomcat') {
            steps {
                script {
                    sh """
                    sudo chmod 400 /var/lib/jenkins/nv_shiv.pem
                    ansible-playbook deploy_tomcat.yml -i hosts.ini --private-key /var/lib/jenkins/nv_shiv.pem -u ubuntu -e 'BUILD_NUMBER=${BUILD_NUMBER}'
                    """
                }
            }
        }
    }
}
