pipeline {
    agent {
        label 'AGENT-1'
    }
    options{
        disableConcurrentBuilds()
        timeout(time:30, unit: 'MINUTES')
    }
    parameters {    //added this params
       booleanParam(name: 'deploy', defaultValue: false, description: 'Select to Deploy or Not')   //toggle button, like switch

        choice(name: 'ENVIRONMENT', choices: ['dev', 'qa', 'uat', 'pre-prod', 'prod'], description: 'Select Your Environment')  //like dropdown
    
        string(name: 'version', description: 'Enter your application version')  //entering like user values 1,2,3, etc
    }   
    environment{
        appVersion = ''
        region = 'us-east-1'
        //account_id = '537124943253'
        project = 'expense'    
        environment = '' //remove here, we will take dynamically using params.
        component  = 'backend'
        account_id = ''
    }

    stages {
        stage('Setup Environment') {    //added this stage
            steps{
                script{
                    environment = params.ENVIRONMENT    //dynamically takes
                    appVersion  = params.version
                    account_id  = pipelineGlobals.getAccountID(environment)
                }
            }
        }
        stage('Integration Test') {    //added this stage
            when{
                expression {params.ENVIRONMENT == 'qa'}
            }
            steps{
                script{
                    sh """
                        echo "RUN integration test"
                    """
                }
            }
        }
        stage('Check JIRA') {    //added this stage
            when{
                expression {params.ENVIRONMENT == 'prod'}
            }
            steps{
                script{
                    sh """
                        echo "check jira status"
                        echo "check jira deployment window"
                        echo "fail pipeline if above two are not true"
                    """
                }
            }
        }
        // write Integration Test
        // Write JIIRA Stage
        stage('Deploy $component'){
            steps{
                withAWS(region: 'us-east-1', credentials: 'aws-creds'){
                    sh """
                    aws eks update-kubeconfig --region ${region} --name ${project}-${environment}

                    cd helm     

                    sed -i 's/IMAGE_VERSION/${appVersion}/g' values-${environment}.yaml     

                    helm upgrade --install ${component} -n ${project} -f values-${environment}.yaml .  

                    """
                    //dot means current folder of Dockerfile exists
                    //sed: Streamline Editor--> without opening the file, we can substitute the values.
                }
            }
        }
    }
    post {
        always{
            echo "This is ALWAYS section, always say hello"
            deleteDir()
        }

        success{
            echo "This section print when pipeline is success, I am SUCCESS"
        }

        failure{
            echo "this section failed when pipeline is FAILED. I am FAIL"
        }
    }
}