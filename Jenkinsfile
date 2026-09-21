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
        stage('Dependabot Security Gate') {
            environment {
                GITHUB_OWNER = 'tmahicharan'
                GITHUB_REPO  = 'catalogue'
                GITHUB_API   = 'https://api.github.com'
                GITHUB_TOKEN = credentials('GITHUB_TOKEN')
            }

            steps {
                script{
                    /* Use sh """ when you want to use Groovy variables inside the shell.
                    Use sh ''' when you want the script to be treated as pure shell. */
                    sh '''
                    echo "Fetching Dependabot alerts..."

                    response=$(curl -s \
                        -H "Authorization: token ${GITHUB_TOKEN}" \
                        -H "Accept: application/vnd.github+json" \
                        "${GITHUB_API}/repos/${GITHUB_OWNER}/${GITHUB_REPO}/dependabot/alerts?per_page=100")

                    echo "${response}" > dependabot_alerts.json

                    high_critical_open_count=$(echo "${response}" | jq '[.[] 
                        | select(
                            .state == "open"
                            and (.security_advisory.severity == "high"
                                or .security_advisory.severity == "critical")
                        )
                    ] | length')

                    echo "Open HIGH/CRITICAL Dependabot alerts: ${high_critical_open_count}"

                    if [ "${high_critical_open_count}" -gt 0 ]; then
                        echo "❌ Blocking pipeline due to OPEN HIGH/CRITICAL Dependabot alerts"
                        echo "Affected dependencies:"
                        echo "$response" | jq '.[] 
                        | select(.state=="open" 
                        and (.security_advisory.severity=="high" 
                        or .security_advisory.severity=="critical"))
                        | {dependency: .dependency.package.name, severity: .security_advisory.severity, advisory: .security_advisory.summary}'
                        exit 1
                    else
                        echo "✅ No OPEN HIGH/CRITICAL Dependabot alerts found"
                    fi
                    '''
                    
                }
            }
        }

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