pipeline {
    agent any
    stages {
        stage ('Build BackEnd') {
            steps {
                bat 'mvn clean package -DskipTests=true'
            }
        }
        stage ('Unit Tests') {
            steps {
                bat 'mvn test'
            }
        }
        stage ('Sonar Analysis') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL') {
                   bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://192.168.1.106:9000 -Dsonar.login=208d9d8b09acf6d8ff52afd474ddc7f13e80433d -Dsonar.java.binaries=target"
                }
            }
        }
        stage ('Deploy BackEnd') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://192.168.1.106:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('API Test') {
            steps {
                dir('api-test') {
                    git credentialsId: 'github_login', url: 'https://github.com/eduardo-hayata/tasks-api-test'
                    bat 'mvn test'
                }
            }
        }
        stage ('Deploy FrontEnd') {
            steps {
                dir('frontend') {
                    git credentialsId: 'github_login', url: 'https://github.com/eduardo-hayata/tasks-frontend'
                    bat 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://192.168.1.106:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }
        stage ('Functional Test') {
            steps {
                dir('functional-test') {
                    git credentialsId: 'github_login', url: 'https://github.com/eduardo-hayata/tasks-functional-tests'
                    bat 'mvn test'
                }
            }
        }
    }
    post {
       always {
           junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml, functional-test/target/surefire-reports/*.xml'
           archiveArtifacts artifacts: 'target/tasks-backend.war, frontend/target/tasks.war', followSymlinks: false, onlyIfSuccessful: true
       }
    }
}
