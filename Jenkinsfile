pipeline {
    agent any
    stages {
        stage ('Build Backend') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }
        stage ('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }
        stage ('Sonar Analysis') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('Sonar'){
                    sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://10.151.83.139:9000 -Dsonar.login=3cd95f55ce1a7cc21cbdf6ab9c87f256d403a729 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }
        }
        stage ('Quality Gate'){
            steps {
                sleep(3)
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ('Deploy Backend') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://10.151.83.127:8001/')], contextPath: '/tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('API Test') {
            steps {
                dir ('api-test'){
                    git 'https://github.com/lmbleandro/tasks-api-test'
                    sh 'mvn test'
                }
            }
        }
        stage ('Deploy FrontEnd') {
            steps {
                dir ('frontend'){
                    git 'https://github.com/lmbleandro/tasks-frontend.git'
                    sh 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://10.151.83.127:8001/')], contextPath: '/tasks', war: 'target/tasks.war'
                }
            }
        }
        stage ('Automate Tests') {
            steps {
                dir ('automate_test'){
                    git 'https://github.com/lmbleandro/tasks-functional-tests.git'
                    sh 'mvn test'
                }
            }
        }
        stage ('Deploy Prod'){
            steps {
                sh 'docker-compose build'
                sh 'docker-compose up -d'
            }
        }
        stage ('Helth Check') {
            steps {
                sleep(5)
                dir ('automate_test'){
                    sh 'mvn verify -Dskip.surefire.tests'
                }
            }
        }
    }
    post{
        always{
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, \
                                                         api-test/target/surefire-reports/*.xml, \
                                                         functional-test/target/surefire-reports/*.xml, \
                                                         functional-test/target/failsafe-reports/*.xml'

        }
        unsuccessful {
            emailext attachLog: true, body: 'See the attached log bellow', subject: 'Build $BUILD_NUMBER has failed', to: 'lmbleandro2010+jenkins@gmail.com'
        }
        fixed {
            emailext attachLog: true, body: 'See the attached log bellow', subject: 'Build is fine!!!', to: 'lmbleandro2010+jenkins@gmail.com'
        }
    }
}