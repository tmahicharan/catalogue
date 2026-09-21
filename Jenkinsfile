pipeline {
    agent {
        label 'AGENT-1'
    }
    environment{
        appversion=""
        ACC_ID= "677673473487"
        PROJECT= "roboshop"
        COMPONENT= "catalogue"
    }
    
    options {
         timeout(time: 10, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }


    stages {
        stage ('Read version') {
            steps{
                script{
                    def PACKAGEJSON = readJSON file: 'package.json'
                    appversion = PACKAGEJSON.version
                    echo "appVersion: ${appversion}"
                } 
              
            }
        }
        stage ('Install dependencies') {
            steps {
                script{
                    sh """
                    npm install
                    """
                }
            }
        }
        stage ('unit Test') {
            steps {
                script{
                    sh """
                      npm test
                    """
                }
            }
        }
         /* stage('Sonar Scan'){
            environment {
                def scannerHome = tool 'sonar-8.0'
            }
            steps {
                script{
                    withSonarQubeEnv('sonar-server') {
                        sh  "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Wait for the quality gate status
                    // abortPipeline: true will fail the Jenkins job if the quality gate is 'FAILED'
                    waitForQualityGate abortPipeline: true 
                }
            }
        } */
        stage('Build Image') {
            steps {
                script{
                    withAWS(region:'us-east-1',credentials:'aws-auth') {
                        sh """
                            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                            docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appversion} .
                            docker images
                            docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appversion}
                        """
                    }
                }
            }
        }
        

    }
    post{
        always {
            echo "I will run even pipeline fail"
            cleanWs()
        }
        success {
            echo "I will run if sucess"
        }
        failure {
            echo "I will run if failure"
        }
        aborted {
            echo "pipeline is aborted"
        }
    }
}